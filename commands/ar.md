# /ar

Manage accounts receivable — create invoices, receive payments, deposit funds, and review what customers owe you.

## Usage
```
/ar                          # Review outstanding invoices and AR aging
/ar invoice <description>    # Create a customer invoice
/ar payment <description>    # Record a customer payment against open invoices
/ar deposit                  # Move funds from Undeposited Funds to bank
/ar credit <customer>        # Issue a credit memo to a customer
/ar refund <description>     # Issue a cash refund to a customer
/ar aging                    # AR aging report with DSO and recommendations
```

## Behavior

Use the **Accountant** agent with the **Accounts Receivable** skill.

### /ar (no args) — Review Outstanding Invoices
1. `qbReports(reportType="AgedReceivables")` — pull summary aging
2. Present totals by bucket: Current, 1-30, 31-60, 61-90, 90+ days
3. Highlight invoices over 90 days (may need bad debt discussion with CPA)
4. Suggest follow-up actions for overdue customers

### /ar invoice — Create an Invoice
1. Parse customer, items/services, amounts, and due date from the description
2. `qbMasterData` — look up customer ID, item/service IDs, and sales terms
3. `agentMemory(operation="read")` — check known customer billing patterns
4. `qbFetchTransactions(transactionType="Invoice", outstandingOnly=true, entityId=customerId)` — duplicate check
5. Show confirmation: customer, line items, total, due date
6. `qbInvoice` — record the invoice
7. Prompt to attach supporting documents via `qbAttachFile(entityType="Invoice")` — recommended for audit trail
8. `agentMemory` — upvote customer mapping

### /ar payment — Receive a Customer Payment
1. `qbFetchTransactions(transactionType="Invoice", outstandingOnly=true, entityId=customerId)` — find open invoices
2. Show outstanding invoices; confirm which to apply the payment to
3. Show confirmation: customer, invoices being paid, total
4. `qbReceivePayment` with customerId, totalAmount, paymentDate, and invoices array
   - Partial payments: the invoice stays partially outstanding
   - Overpayments: QB creates an unapplied credit (shown in the response as `unappliedAmount`)
5. Prompt to attach remittance advice via `qbAttachFile(entityType="Payment")` — recommended for audit trail
6. `agentMemory` — upvote customer mapping

> Note: `qbReceivePayment` moves funds to Undeposited Funds, NOT directly to the bank. Run `/ar deposit` to complete the bank deposit.

### /ar deposit — Deposit to Bank
1. `qbFetchTransactions` on Undeposited Funds account to see pending items
2. Show the user which payments are waiting and their total
3. Confirm which payments to group into this deposit (match to the bank statement lump sum)
4. `qbDeposit` with depositAccountId (the bank account) and line items from Undeposited Funds
5. Deposit total should match the bank statement line

### /ar credit — Issue a Credit Memo
1. `qbMasterData` — look up customer ID and relevant account/item IDs
2. `qbCredit(creditType="customer")` with customerId, txnDate, and lines
3. To apply: include the credit memo in `creditMemos` array when calling `qbReceivePayment`

### /ar refund — Issue a Cash Refund
1. Parse customer, amount, and refund date from the description
2. `qbMasterData` — look up customer ID, item IDs, and bank account to refund from
3. `qbFetchTransactions(transactionType="RefundReceipt", entityId=customerId, startDate, endDate)` — duplicate check
4. Show confirmation: customer, amount, refund account
5. `qbRefundReceipt` with customerId, depositAccountId (bank refund comes from), txnDate, and lines

### /ar aging — Aging Report with Analysis
Delegates to `/aging ar` — same report, same DSO calculation, same recommendations. Use `/aging ar` directly for aging analysis.

## The AR Flow
```
Invoice → ReceivePayment → Undeposited Funds → Deposit → Bank Account
```
For immediate cash sales with no invoice: `qbSalesReceipt` → Undeposited Funds → `qbDeposit`.
To record an immediate cash sale, use `/record` instead.

## Safety
- NEVER skip the duplicate check before creating an invoice
- NEVER use `qbDeposit` to record a customer payment when an outstanding invoice exists — use `qbReceivePayment` first, then deposit
- `qbReceivePayment` does NOT put money in the bank — always follow with `qbDeposit` to match the bank statement
- Always confirm with the user before recording any transaction
