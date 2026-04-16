---
name: bank-feed-processing
description: Process bank feed transactions — categorize, match, record, or flag for review using agent memory and confidence scoring. Use when the user mentions bank feed, bank transactions, categorize transactions, or auto-categorize.
---

# Bank Feed Processing Skill

Process unrecorded bank and credit card transactions from the connected bank feed. Categorize using agent memory, match to existing records, and flag uncertain items for CPA review.

## Trigger

Activate when the user wants to:
- Process bank feed transactions
- Categorize bank transactions
- Review unrecorded transactions
- Auto-categorize using learned patterns
- Flag transactions for CPA review

## Prerequisites

### Bootstrap Check
Before processing, verify: `agentMemory(operation="read", type="worklog")`
- If client is NOT bootstrapped → warn: "This client hasn't been bootstrapped — most vendors are unknown. Run bootstrap first for better accuracy."
- Without bootstrap, most transactions will be flagged (low confidence).

## The Confidence Model

Each bank feed transaction is evaluated against agent memory:

| Upvotes | Confidence | Action |
|---------|------------|--------|
| 5+ | High | Record directly with top-voted account |
| 3-4 | Medium | Record with note, mention categorization |
| 1-2 | Low | Proceed with caution, consider flagging |
| 0 | None | Flag for CPA review |

Confidence comes from: bootstrap seeding (capped at 5), CPA approvals, and real-time upvotes after each successful recording.

## Workflow: Process Bank Feed

### Step 1: Fetch Transactions
```
bankFeed(action="fetch")
```
Returns unprocessed transactions enriched with agent memory matches, document links, and suggested categories.

### Step 2: Evaluate Each Transaction

For each transaction:

1. **Check enrichment** — Does the bank feed response include a memory match?
2. **Check for existing records** — Before recording:
   - Expenses/debits: `qbFetchTransactions(transactionType="Bill", outstandingOnly=true, entityId=vendorId)` → if outstanding bill exists, use `qbBillPayment` not `qbExpense`
   - Deposits/credits: `qbFetchTransactions(transactionType="Invoice", outstandingOnly=true, entityId=customerId)` → if outstanding invoice exists, use `qbReceivePayment` not `qbDeposit`
3. **Duplicate check** — `qbFetchTransactions` with vendor + date (±3 days) + amount
4. **Anomaly check** — If amount is 3x outside the learned range for this vendor, flag regardless of confidence

### Step 3: Record or Flag

**High confidence (5+ upvotes):**
1. `qbMasterData` — lookup IDs
2. Record with the top-voted account mapping
3. `agentMemory` — upvote the mapping
4. `fetchWorkQueue(source="markRecorded")` — prevent re-processing

**Medium confidence (3-4 upvotes):**
1. Same as high, but include a note in the response about the categorization
2. Upvote on success

**Low/No confidence:**
1. `flagForReview` with specific `aiReasoning`:
   - "New vendor not in memory"
   - "Amount $X is 3x the usual $Y for this vendor"
   - "Multiple possible categories: [list]"
   - "Description is ambiguous: [description]"
2. Include `suggestedCategory` when you have a reasonable guess

### Step 4: Batch When Possible

When 3+ transactions share the same type and source account:
1. Group by transaction type (Expense, Deposit, etc.)
2. Run one `qbMasterData` lookup for all vendor/customer IDs
3. Run one `qbFetchTransactions` duplicate check covering the full date range
4. Submit via `qbBatch`
5. Check response for per-item success/failure
6. Retry failed items individually

### Step 5: Report Summary

Present results:
```
Bank Feed Summary — [Date]
═══════════════════════════
Processed: X transactions
Recorded:  Y ($total)
Flagged:   Z for CPA review
Skipped:   W (duplicates)
```

Include a table of flagged items with reasoning.

## Workflow: Review Mode

When the user wants to see transactions before recording:

1. `bankFeed(action="fetch")` — pull all unprocessed
2. Present in a table with: date, description, amount, suggested category, confidence level
3. Let the user pick which to process
4. Do NOT record anything until explicitly told

## Workflow: Flag a Specific Transaction

When the user wants to flag a particular transaction:

1. Identify the transaction by description, amount, or date
2. `flagForReview(tellerTransactionId=..., aiReasoning=...)` with the user's reason
3. Confirm: "Flagged for CPA review with reason: [reason]"

## Workflow: Process CPA-Approved Items

CPA-approved items from the review queue take priority:

1. `fetchWorkQueue(source="approvedReviews")` — get approved items
2. Record using the `effectiveCategory` from the CPA's approval
3. Mark recorded: `fetchWorkQueue(source="markRecorded", reviewItemNumber=N, qbTransactionId=ID)`
4. **Never override** a CPA-approved category with agent judgment

## Safety Checklist

- [ ] Bootstrap status checked before processing
- [ ] Outstanding bills/invoices checked before recording expenses/deposits
- [ ] Duplicate check run for every transaction
- [ ] Anomaly check (3x outside learned range) applied
- [ ] CPA-approved items processed first and categories preserved
- [ ] Agent memory upvoted after each successful recording
- [ ] Transactions marked as recorded to prevent re-processing

## Common Mistakes to Avoid

- Recording an expense when an outstanding bill exists for the same vendor → use BillPayment
- Recording a deposit when an outstanding invoice exists → use ReceivePayment
- Skipping the bootstrap check → floods the review queue with flags
- Not marking transactions as recorded → they appear again in the next fetch
- Overriding a CPA-approved category with a different agent guess
- Ignoring amount anomalies just because the vendor has high upvotes
