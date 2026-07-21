---
name: bank-feed-processing
description: Process bank feed transactions — categorize, match, record, or flag for review using QB history and the consistency rule. Use when the user mentions bank feed, bank transactions, categorize transactions, or auto-categorize.
---

# Bank Feed Processing Skill

Process unrecorded bank and credit card transactions from the connected bank feed. Categorize using QB history, match to existing records, and flag uncertain items for CPA review.

## Trigger

Activate when the user wants to:
- Process bank feed transactions
- Categorize bank transactions
- Review unrecorded transactions
- Auto-categorize using learned patterns
- Flag transactions for CPA review

## Prerequisites

### Bootstrap Check
Before processing, verify: `agentMemory(operation="read", type="system", title="bootstrap_status")`
- If client is NOT bootstrapped → warn: "This client hasn't been bootstrapped — most vendors are unknown. Run bootstrap first for better accuracy."
- Without bootstrap, most transactions will be flagged (no matching history).

## The Decide Gate

Record a transaction directly ONLY when one of these holds:

| Condition | Source | Action |
|-----------|--------|--------|
| CPA approved via task | `tasks(operation="list")` → `effectiveCategory` | Record verbatim, complete the task |
| Outstanding document exists | `qbFetchTransactions(outstandingOnly=true)` | `qbBillPayment` / `qbReceivePayment` — the document already encodes the account |
| History-consistent | `qbFetchTransactions` 6-month entity scan | Record with the dominant account |

**Consistency rule**: use the dominant account ONLY IF ≥3 transactions, dominant share ≥70%, no runner-up ≥20%, and amount within 5× the median.

Anything else → flag for CPA review (`tasks` create). Never guess an account. User confirmation is reserved for interrupts: potential duplicates, amount anomalies, wrong-type guards, voids.

## Workflow: Process Bank Feed

### Step 1: Fetch Transactions
```
bankFeed(action="fetch")
```
Returns unprocessed transactions enriched with agent memory matches, document links, and suggested categories.

### Step 2: Evaluate Each Transaction

For each transaction:

1. **Check `alreadyFlagged`** — a task already exists for this transaction → skip it (it's the CPA's turn)
2. **Check for existing records** — Before recording:
   - Expenses/debits: `qbFetchTransactions(transactionType="Bill", outstandingOnly=true, entityId=vendorId)` → if outstanding bill exists, use `qbBillPayment` not `qbExpense`
   - Deposits/credits: `qbFetchTransactions(transactionType="Invoice", outstandingOnly=true, entityId=customerId)` → if outstanding invoice exists, use `qbReceivePayment` not `qbDeposit`
3. **Duplicate check** — `qbFetchTransactions` with vendor + date (±15 days) + amount
4. **Anomaly check** — If amount is 3x outside the historical range for this vendor, flag even if the gate passes

### Step 3: Record or Flag

**Gate passes (CPA-approved, document match, or history-consistent):**
1. `qbMasterData` — lookup IDs
2. Record — the CPA's `effectiveCategory` verbatim, the outstanding document's account, or the dominant historical account
3. `agentMemory` — write a `patterns` entry if the charge is now confirmed recurring (2+ occurrences)
4. `tasks(operation="complete", taskNumber, qbTransactionId)` — prevent re-processing

**Gate fails:**
1. `tasks(operation="create")` with specific `aiReasoning`:
   - "New vendor not in memory"
   - "Amount $X is 3x the usual $Y for this vendor"
   - "Multiple possible categories: [list]"
   - "Description is ambiguous: [description]"
2. Include `suggestedCategory` when you have a reasonable guess

### Step 4: Handling Multiple Transactions

There is no batch tool — each transaction is recorded with its own tool call. When 3+ transactions share the same type and source account, save round-trips on the lookups:
1. Group by transaction type (Expense, Deposit, etc.)
2. Run one `qbMasterData` lookup for all vendor/customer IDs
3. Run one `qbFetchTransactions` duplicate check covering the full date range
4. Record each transaction individually with the appropriate tool (`qbExpense`, `qbDeposit`, etc.)
5. Track per-item success/failure
6. Retry failed items; do not let one failure stop the rest

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
2. Present in a table with: date, description, amount, suggested category, gate outcome (record / flag)
3. Let the user pick which to process
4. Do NOT record anything until explicitly told

## Workflow: Flag a Specific Transaction

When the user wants to flag a particular transaction:

1. Identify the transaction by description, amount, or date
2. `tasks(operation="create", tellerTransactionId=..., aiReasoning=...)` with the user's reason
3. Confirm: "Flagged for CPA review with reason: [reason]"

## Workflow: Process CPA-Approved Items

CPA-approved tasks take priority:

1. `tasks(operation="list")` — get open tasks assigned to the AI agent
2. Record using the `effectiveCategory` from the CPA's approval
3. Complete: `tasks(operation="complete", taskNumber=N, qbTransactionId=ID)`
4. **Never override** a CPA-approved category with agent judgment

## Safety Checklist

- [ ] Bootstrap status checked before processing
- [ ] Outstanding bills/invoices checked before recording expenses/deposits
- [ ] Duplicate check run for every transaction
- [ ] Anomaly check (3x outside learned range) applied
- [ ] CPA-approved items processed first and categories preserved
- [ ] Newly confirmed recurring charges written as `patterns` entries
- [ ] Transactions marked as recorded to prevent re-processing

## Common Mistakes to Avoid

- Recording an expense when an outstanding bill exists for the same vendor → use BillPayment
- Recording a deposit when an outstanding invoice exists → use ReceivePayment
- Skipping the bootstrap check → floods the CPA task list with escalations
- Not marking transactions as recorded → they appear again in the next fetch
- Overriding a CPA-approved category with a different agent guess
- Ignoring amount anomalies just because the vendor has strong history
