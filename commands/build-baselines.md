---
description: Build or refresh vendor baselines in agent memory from QuickBooks transaction history — required for effective anomaly detection
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [days|"full"]
---

Build structured vendor baselines from QuickBooks transaction history and store them in agent memory. These baselines power anomaly detection, auto-categorization confidence, and vendor spend analysis.

Reference: Read the vendor baseline model at `skills/anomaly-detection/references/vendor-baseline-model.md` for the full data schema and computation rules.

If $ARGUMENTS is "full", scan last 180 days for comprehensive baselines.
If $ARGUMENTS is a number, use that as the look-back period in days.
Default: last 90 days.

Execute autonomously — no intermediate questions.

## Steps

### 1. Check for Existing Baselines
```
agentMemory(operation: "read", category: "vendor_baseline")
```
- If baselines exist → this is an UPDATE scan (merge new data into existing)
- If no baselines → this is a FIRST BUILD (create from scratch)
- Note the count of existing baselines for the report

### 2. Determine Date Range
- For FIRST BUILD: use the full look-back period (default 90 days)
- For UPDATE: use the most recent `last_seen` date among existing baselines as start date
  - Minimum: at least 7 days of new data to make an update worthwhile
  - If less than 7 days since last update → skip with message "Baselines are already current"

### 3. Fetch Transactions
Pull all transaction types that create vendor touchpoints:
```
qbFetchTransactions(transactionType: "Purchase", startDate: <start>, endDate: <today>, maxResults: 500)
qbFetchTransactions(transactionType: "Bill", startDate: <start>, endDate: <today>, maxResults: 500)
```

If the transaction count seems truncated at 500, note this in the report. Consider narrowing the date range for accuracy.

### 4. Group by Vendor
- Extract vendor from `VendorRef.name` (for Bills) or entity name (for Purchases)
- Normalize: trim whitespace, lowercase for subject key, preserve original case in vendor_name
- Skip transactions with no identifiable vendor

### 5. Compute Statistics per Vendor
For each vendor with ≥1 transaction:

**Amount statistics:**
- min, max, mean (average), median
- stddev (standard deviation) — use 0 if only 1 transaction
- p95 (95th percentile) — use max if <20 transactions

**Frequency analysis:**
- Sort transactions by date
- Compute days between consecutive transactions
- Classify pattern: monthly, weekly, biweekly, quarterly, annual, irregular
- avg_days_between, min_days_between, max_days_between
- transactions_per_month = transaction_count / (date_range_days / 30)

**Categorization:**
- Count which QB account each transaction was posted to
- primary_account = most common account, primary_pct = its percentage
- secondary_account = next most common (if <100% primary), secondary_pct

**Flags:**
- is_recurring: frequency != "irregular" AND transaction_count >= 3
- is_single_purpose: primary_pct >= 95
- has_round_amounts: >50% of amounts end in .00 and are >$500
- weekend_transactions_pct: % of transactions on Saturday/Sunday
- seasonal_pattern: null (needs 12+ months to detect)

### 6. Save/Update Baselines

**For new vendors (no existing baseline):**
```
agentMemory(
  operation: "write",
  organizationId: <org_id>,
  category: "vendor_baseline",
  subject: "baseline:<vendor_name_lowercase>",
  memory: "<full JSON per schema>",
  confidence: 60
)
```

**For existing vendors (update):**
- Merge new transactions into existing stats
- Recompute all amount statistics from combined data
- Update frequency metrics, last_seen, transaction_count, scan_count (+1)
- Increase confidence: 60→75 on second scan, cap at 90
```
agentMemory(
  operation: "update",
  memoryId: <existing_memory_id>,
  memory: "<updated JSON>",
  confidence: <new_confidence>
)
```

### 7. Report

Output a summary report:

```
## Vendor Baseline Report — [Date]
**Client:** [Org Name]
**Scan Period:** [start] to [end]
**Transactions Analyzed:** [N]

### Baselines Created: [count new]
| Vendor | Txns | Avg Amount | Frequency | Primary Category |
|--------|------|-----------|-----------|-----------------|
| ...    | ...  | ...       | ...       | ...             |

### Baselines Updated: [count updated]
| Vendor | New Txns | Prev Avg → New Avg | Confidence |
|--------|---------|-------------------|-----------|
| ...    | ...     | ...               | ...       |

### Vendors with Minimal Data (1 transaction only): [count]
- [List vendor names — baselines created with wide tolerance ranges]

### Coverage
- Vendors with baselines: [N]
- Vendor coverage of scan period spend: [%] (sum of baselined vendor spend / total spend)

**Next recommended build:** [7 days from now or before next anomaly scan]
```

## Important Rules
1. **Respect API limits** — 500 results per fetch call. If hitting the limit, note it.
2. **Don't overwrite human corrections** — if a baseline has confidence ≥85 (human-validated), preserve its anomaly_history and only update amount/frequency stats.
3. **Handle edge cases** — vendors with 1 transaction get minimal baselines. Vendors with identical amounts get stddev=0 (note: anomaly detection should use 10% threshold instead).
4. **Memory format must be valid JSON** — always validate the JSON string before writing.
