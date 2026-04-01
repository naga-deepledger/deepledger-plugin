# Bookkeeping Skill

Expertise in recording financial transactions in QuickBooks Online, processing bank feeds, reconciling accounts, and closing the books at month-end.

## Trigger

Activate this skill when the user wants to:
- Record any financial transaction (expense, bill, invoice, payment, deposit, journal entry, transfer)
- Process bank feed transactions
- Reconcile a bank or credit card account
- Close the books for a month
- Look up or create vendors, customers, or accounts
- Attach documents to transactions
- Check for duplicate transactions

## Workflow: Recording a Transaction

Follow the `getGuide(guideType="transaction_recording")` workflow:

### 1. Identify Transaction Type
Ask or determine: Is this an expense, bill, invoice, deposit, payment, transfer, or journal entry?

| Situation | Tool |
|-----------|------|
| Paid vendor immediately (bank/card) | `qbExpense` |
| Vendor bill to pay later | `qbBill` |
| Paying an outstanding bill | `qbBillPayment` |
| Customer owes you | `qbInvoice` |
| Customer paid on the spot | `qbSalesReceipt` |
| Receiving payment on invoice | `qbReceivePayment` |
| Bank deposit | `qbDeposit` |
| Adjusting entry | `qbJournalEntry` |
| Between own accounts | `qbTransfer` |
| Refund to customer | `qbRefundReceipt` |
| Applying credit | `qbCredit` |

### 2. Master Data Lookup
```
qbMasterData → get vendor/customer ID, account IDs, item IDs
agentMemory  → check for known vendor-to-account mappings
```

If vendor/customer doesn't exist, create them via `qbMasterData`.

### 3. Duplicate Check (MANDATORY)
```
qbFetchTransactions → filter by vendor + date (±3 days) + amount range
```
- Use `outstandingOnly=true` for bills and invoices
- Use `entityId` to filter by vendor/customer
- Use `minAmount`/`maxAmount` to match the amount

### 4. Record the Transaction
- Include `vendorId`/`customerId` for audit trail
- Source account (bank/CC) must differ from line item account (category)
- For Journal Entries: debits MUST equal credits
- Use `memo` for internal notes, `docNumber` for external references

### 5. Attach Supporting Documents
```
qbGetUploadUrl → get pre-signed URL, then execute the curl command
```
Note: entityType for expenses = "Purchase" (not "Expense")

### 6. Update Agent Memory
```
agentMemory → write/upvote vendor-to-account mapping
```

## Workflow: Bank Feed Processing

Follow the `getGuide(guideType="reconciliation")` workflow:

1. `bankFeed(action="fetch")` — get enriched transactions with memory context
2. For each transaction, evaluate confidence:
   - **High** (3+ upvotes in memory) → record directly
   - **Medium** (1-2 upvotes) → record with note
   - **Low** (no match) → `bankFeed(action="flag")` for CPA review
3. Check for outstanding Bills/Invoices before recording Expenses/Deposits
4. After recording, `agentMemory` upvote the account used
5. `fetchWorkQueue(source="markRecorded")` to prevent re-processing

## Workflow: Month-End Close

Follow the `getGuide(guideType="month_end_closing")` workflow:

1. Reconcile all bank & CC accounts (`qbReconciliationCheck`)
2. Review outstanding AP/AR (`qbReports` → AgedPayables, AgedReceivables)
3. Record accruals & deferrals (`qbJournalEntry`)
4. Record depreciation (`qbJournalEntry`)
5. Review financial statements (`qbReports` → P&L, Balance Sheet, Cash Flow)
6. Final trial balance check (`qbReports` → TrialBalance)

Target: health score >= 90 on all accounts before closing.

## Safety Checklist

- [ ] `qbMasterData` lookup completed — have valid IDs
- [ ] Duplicate check via `qbFetchTransactions` — no match exists
- [ ] Source account != category account on expense lines
- [ ] Journal Entry debits = credits
- [ ] User confirmation obtained before creating the transaction
- [ ] Agent memory updated after successful recording

## Common Mistakes to Avoid

- Recording an Expense when an outstanding Bill exists → use `qbBillPayment`
- Recording a Deposit when an outstanding Invoice exists → use `qbReceivePayment`
- Forgetting `vendorId`/`customerId` → breaks audit trail
- Wrong `entityType` for attachments (Expense = "Purchase" in API)
- Creating a Bill Payment without checking for outstanding bills first
