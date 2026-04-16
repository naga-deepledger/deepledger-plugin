---
name: transaction-recording
description: Record any financial transaction in QuickBooks — expenses, bills, invoices, deposits, payments, journal entries, and transfers. Use when the user mentions recording, entering, posting, or booking a transaction.
---

# Transaction Recording Skill

Record any financial transaction in QuickBooks Online. Handles the full decision tree from identifying the transaction type to writing it, with mandatory safety checks at every step.

## Trigger

Activate when the user wants to:
- Record, enter, post, or book a transaction
- Log an expense, bill, invoice, deposit, payment, or transfer
- Make a journal entry or adjusting entry
- Correct or reverse a transaction
- Batch-record multiple transactions

## Transaction Type Decision Tree

### Money Going OUT (Expense Side)

| Situation | Tool |
|-----------|------|
| Paid vendor immediately (bank/card) | `qbExpense` |
| Vendor bill to pay later | `qbBill` |
| Paying an outstanding bill | `qbBillPayment` |

### Money Coming IN (Income Side)

| Situation | Tool |
|-----------|------|
| Customer owes you (pay later) | `qbInvoice` |
| Customer paid on the spot (no invoice) | `qbSalesReceipt` |
| Customer paying an outstanding invoice | `qbReceivePayment` |

### Money Hitting the BANK (Deposits)

| Situation | Tool |
|-----------|------|
| Batching payments from Undeposited Funds | `qbDeposit` (LinkedTxn) |
| Non-customer income (interest, refunds, contributions) | `qbDeposit` (Direct) |

### Other

| Situation | Tool |
|-----------|------|
| Adjusting / reclassifying entry | `qbJournalEntry` |
| Between own accounts | `qbTransfer` |
| Refund to customer | `qbRefundReceipt` |
| Applying a credit | `qbCredit` |

## Workflow: Record Any Transaction

### Step 1: Identify Type
Parse the user's description. If unclear, ask. The decision tree above determines the correct tool.

### Step 2: Master Data Lookup
```
qbMasterData → vendor/customer ID, account IDs, item IDs
agentMemory  → known vendor-to-account mappings
```
If vendor/customer doesn't exist, create via `qbMasterData(operation="create")`.

### Step 3: Duplicate Check (MANDATORY)
```
qbFetchTransactions → filter by vendor/customer + date (±3 days) + amount range
```
- Use `outstandingOnly=true` for bills and invoices
- Use `entityId` for vendor/customer filtering
- Use `minAmount`/`maxAmount` to bracket the amount

### Step 4: Confirm with User
Show what will be recorded: type, vendor/customer, amount, date, accounts, memo. Wait for explicit "yes."

### Step 5: Record
Call the appropriate QB tool with all required fields:
- Always include `vendorId`/`customerId` for audit trail
- Source account must differ from line item account
- Journal Entries: total debits MUST equal total credits
- Use `memo` for internal notes, `docNumber` for external references

### Step 6: Attach Documents (if available)
```
qbAttachFile → get pre-signed URL, then execute the curl command
```
Note: entityType for expenses = "Purchase" (not "Expense" in the API).

### Step 7: Update Memory
```
agentMemory → write new mapping or upvote existing vendor-to-account mapping
```

## Workflow: Batch Recording

Use `qbBatch` when recording 3+ transactions of the same type:

1. `qbMasterData` — lookup all IDs needed
2. `qbFetchTransactions` — single duplicate check covering the full date range
3. Build batch payload — group by transaction type (cannot mix types in one batch)
4. Confirm — show summary: "Recording X transactions totaling $Y"
5. `qbBatch` — submit
6. Verify — check each item for success/failure
7. Retry failed items individually
8. `agentMemory` — upvote all vendor mappings used

## Workflow: Corrections & Reversals

### Void and Re-record (same period, simple error)
1. `qbFetchTransactions` — find the incorrect transaction
2. `qbVoidTransaction` — void it (preserves audit trail)
3. Record the correct transaction

### Reversing Journal Entry (prior period, books closed)
1. Create JE reversing the original (swap debits/credits)
2. Record the correct entry in current period
3. Memo: "Correction of [original transaction ref]"

### When to Use Which
- **Same period, simple error** → Void and re-record
- **Prior period, already closed** → Reversing JE
- **Partial correction** → Reversing JE for incorrect portion only

## Safety Checklist

- [ ] `qbMasterData` lookup completed — valid IDs
- [ ] Duplicate check via `qbFetchTransactions` — no match exists
- [ ] Source account != category account on expense lines
- [ ] Journal Entry debits = credits
- [ ] User confirmation obtained before creating
- [ ] Agent memory updated after successful recording

## Common Mistakes to Avoid

- Recording an Expense when an outstanding Bill exists → use `qbBillPayment`
- Recording a Deposit when an outstanding Invoice exists → use `qbReceivePayment`
- Forgetting `vendorId`/`customerId` → breaks audit trail
- Wrong `entityType` for attachments (Expense = "Purchase")
- Source account same as category account → accounting error
- JE debits != credits → QB will reject it
- Skipping duplicate check → hours to fix a double-posted transaction
