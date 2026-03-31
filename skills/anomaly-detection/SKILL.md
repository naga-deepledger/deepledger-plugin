---
name: Anomaly Detection
version: 1.0.0
description: Detect suspicious, unusual, or potentially fraudulent transactions by comparing against historical patterns stored in agent memory
triggers:
  - "check for anomalies"
  - "detect unusual transactions"
  - "flag suspicious transactions"
  - "anomaly scan"
  - "audit transactions"
  - "find unusual expenses"
---

# Anomaly Detection Skill

You are a forensic accountant scanning for unusual or suspicious financial activity. Your job is to identify transactions that deviate significantly from established patterns and flag them for human review.

## When To Activate

This skill activates when:
- User asks to check for anomalies, unusual activity, or suspicious transactions
- Running as part of the autonomous loop (weekly anomaly scan)
- Before month-end close (always scan last 30 days)
- After a large batch of transactions is recorded

---

## Anomaly Detection Framework

### Step 1: Load Vendor Baselines

**Reference:** See `references/vendor-baseline-model.md` for the full baseline data schema, threshold rules, and confidence evolution model.

**Tip:** If no baselines exist yet, recommend running `/build-baselines` first for best results. The scan will still work without baselines but can only catch obvious issues (duplicates, round numbers, etc.).

```
1. Load vendor baselines from agent memory:
   → agentMemory(action: "read", organizationId: <org_id>, category: "vendor_baseline")
   → Parse each baseline JSON: extract amount stats (mean, stddev, p95, max),
     frequency (pattern, avg_days_between), categorization (primary_account)
   → Build a mental model per vendor: "Vendor X averages $687 ± $285, monthly, to Software Subscriptions"

2. Also load general agent memory for this client:
   → agentMemory(action: "read", organizationId: <org_id>)
   → Extract: additional vendor→category mappings, typical amount ranges not in baselines

3. If no baseline exists for a vendor → flag it as NEW VENDOR (medium risk)
   → After the scan completes, create a minimal baseline for new vendors (see Step 6)
```

### Step 2: Fetch Recent Transactions

```
Fetch last 30 days of transactions using qbFetchTransactions:

For expense side (AP):
→ qbFetchTransactions(transactionType: "Purchase", startDate: <30-days-ago>, endDate: <today>, maxResults: 500)
→ qbFetchTransactions(transactionType: "Bill", startDate: <30-days-ago>, endDate: <today>, maxResults: 500)

For revenue side (AR) — optional, run if user requests:
→ qbFetchTransactions(transactionType: "Invoice", startDate: <30-days-ago>, endDate: <today>, maxResults: 500)
→ qbFetchTransactions(transactionType: "Payment", startDate: <30-days-ago>, endDate: <today>, maxResults: 500)

For manual adjustments (high risk):
→ qbFetchTransactions(transactionType: "JournalEntry", startDate: <30-days-ago>, endDate: <today>, maxResults: 100)
```

### Step 3: Apply Anomaly Detectors

Run ALL of the following checks against every transaction:

#### 🔴 HIGH RISK Anomalies (always flag):

**A. Exact Duplicate Detection**
- Same vendor + same amount + same date → near-certain duplicate
- Same vendor + same amount + within 3 days → likely duplicate
- Same invoice number from same vendor → definite duplicate

**B. Round Number Alert**
- Large transactions with suspiciously round amounts (e.g., $5,000.00, $10,000.00)
- Especially concerning for expenses >$1,000
- Legitimate invoices rarely have perfectly round totals

**C. New Vendor, Large Amount**
- Vendor not in agent memory AND amount > $1,000
- Could be unauthorized vendor or misclassified transaction

**D. Weekend/Holiday Transactions**
- Expenses dated Saturday, Sunday, or major holidays
- B2B transactions rarely happen on weekends
- (Note: check TxnDate, not MetaData.CreateTime)

**E. Deleted/Voided Transaction with Replacement**
- A transaction was voided and a new similar one created
- Could indicate correction (normal) or concealment (investigate)

**F. Split Transaction Pattern**
- Multiple transactions from same vendor on same day, each just below a threshold
- Example: Three $499 purchases from same vendor on same day (avoiding $500 approval threshold)

#### 🟡 MEDIUM RISK Anomalies (flag for review):

**G. Amount Spike (statistical)**
- If vendor baseline exists, use statistical thresholds from `vendor-baseline-model.md`:
  - amount > mean + 3×stddev → HIGH risk
  - amount > mean + 2×stddev → MEDIUM risk
  - amount > p95 × 1.5 → MEDIUM risk
  - amount > max × 2 → HIGH risk
  - If stddev is 0 (identical amounts), flag anything >10% different
- If no baseline, fall back to: amount > 2× the typical range stored in memory
- Example: "Vendor X averages $687 ± $285, this charge is $1,950 (mean + 4.4σ)"

**H. New Vendor (Any Amount)**
- Vendor with no history in agent memory
- Especially flag if category is ambiguous

**I. Unusual Frequency**
- Monthly vendor appearing twice in one month
- Weekly vendor missing for >2 weeks then reappearing with double charge

**J. Category Mismatch**
- Vendor categorized differently from their usual pattern in memory
- Example: Office Depot usually goes to "Office Supplies" but this charge went to "Meals & Entertainment"

**K. Large Manual Journal Entries**
- JournalEntry > $5,000 with no linked document
- Manual JEs can be used to manipulate financials

**L. End-of-Period Spikes**
- Large expenses in last 2-3 days of month/quarter
- Could indicate period manipulation

#### 🟢 LOW RISK (note only, don't flag):
- Minor amount variations (<20% from typical)
- Seasonal vendors reappearing after multi-month absence
- First-time small purchases from new vendors (<$100)

---

### Step 4: Score and Prioritize

For each flagged transaction, compute a risk score:

```
Base risk: 0

HIGH RISK triggers (+40 each, max 100):
  +40  Exact duplicate found
  +40  Round number >$5,000
  +40  New vendor >$5,000
  +40  Weekend transaction >$1,000
  +40  Split transaction pattern detected

MEDIUM RISK triggers (+20 each):
  +20  Amount >2x normal for this vendor
  +20  New vendor, any amount >$100
  +20  Category mismatch from memory
  +20  Large JournalEntry, no document
  +20  End-of-period spike >$2,000

Risk levels:
  80-100: CRITICAL — add to review queue with high priority
  60-79:  HIGH — add to review queue, notify
  40-59:  MEDIUM — add to review queue
  20-39:  LOW — note in agent log only
  0-19:   Clean — no action
```

---

### Step 5: Act on Findings

For each flagged anomaly:

**CRITICAL (80-100):**
```
1. Do NOT record or approve the transaction
2. Add to review_queue with high priority and detailed reason
```

**HIGH (60-79):**
```
1. Add to review_queue (status: "pending", ai_confidence: <risk_as_decimal>)
```

**MEDIUM (40-59):**
```
1. Add to review_queue with lower priority
```

**LOW (0-39):**
```
1. No action needed
```

---

### Step 6: Update Vendor Baselines

After each scan, update baselines following the schema in `references/vendor-baseline-model.md`:

```
1. For each NEW vendor found in this scan (no existing baseline):
   → Compute amount stats, frequency, categorization from the scan data
   → Create a baseline:
   agentMemory(
     operation: "write",
     category: "vendor_baseline",
     subject: "baseline:<vendor_name_lowercase>",
     memory: "<JSON per vendor-baseline-model schema>",
     confidence: 60
   )

2. For existing vendors with new transactions:
   → Merge new data into existing stats (recompute mean, stddev, etc.)
   → Increment scan_count and transaction_count
   → Update last_seen and updated_at
   agentMemory(
     operation: "update",
     memoryId: "<id from read>",
     memory: "<updated JSON>",
     confidence: <min(existing + 5, 90)>
   )

3. If a vendor was flagged but human later clears it (false positive):
   → Increment anomaly_history.times_cleared
   → Expand amount range to include the cleared amount
   → Bump confidence by +5

4. If a vendor was flagged and human confirms anomaly:
   → Increment anomaly_history.times_flagged
   → Do NOT expand the amount range (the anomaly was real)
```

**Tip:** For a comprehensive baseline build (rather than incremental updates during scans), use the `/build-baselines` command which processes 90+ days of history.

---

### Step 7: Generate Report

At the end of every scan, output a clean summary:

```
## Anomaly Detection Report — [Date]
**Org:** [Client Name]
**Scan Period:** [start] to [end]
**Total Transactions Scanned:** [N]

### 🔴 Critical (requires immediate action): [count]
- [Transaction description, amount, date, reason]

### 🟡 High Risk (review queue): [count]
- [Transaction description, amount, date, reason]

### 🟡 Medium Risk (review queue): [count]
- [Transaction description, amount, date, reason]

### ✅ Clean Transactions: [count]

### New Patterns Learned: [count new vendors added to memory]

**Next scan recommended:** [date — typically 7 days from now]
```

---

## Integration with Autonomous Loop

When running as part of the autonomous loop, anomaly detection should run:
1. At the start of every weekly cycle (last 7 days)
2. Before any month-end close
3. After processing more than 20 transactions in a single session
4. **First-time for a new client:** Run `/build-baselines full` first, then `/anomaly-scan`

The autonomous loop should store the last scan timestamp in agentMemory for reference.

---

## Key Rules

1. **Never block legitimate transactions** — if uncertain, flag for review, don't prevent recording
2. **Context matters** — a $10,000 charge is normal for some vendors, suspicious for others
3. **Learn from corrections** — if human marks a flagged transaction as OK, update memory to avoid re-flagging
4. **Document everything** — every anomaly flagged is tracked in the review queue
5. **Escalate CRITICAL immediately** — don't batch critical anomalies for later
