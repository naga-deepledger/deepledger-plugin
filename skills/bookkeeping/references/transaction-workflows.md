# Transaction Workflows — Detailed Recording Procedures

## Accounts Payable (AP) Workflows

### Recording a Vendor Bill (qbBill)

Use when: Received a bill/invoice from a vendor that has NOT been paid yet.

1. Look up vendor via qbMasterData (type: "Vendor", search by name)
2. If vendor not found, ask user — create new vendor if confirmed
3. Look up expense account via qbMasterData (type: "Account")
4. Check for duplicates: qbFetchTransactions with vendor name, date, amount
5. Propose bill with: vendor, date, due date, line items, accounts, total
6. On confirmation, call qbBill with:
   - vendorId (from master data lookup)
   - txnDate (bill date)
   - dueDate (if known)
   - lines: array of { amount, accountId, description }
   - Optional: referenceNumber, memo, class

### Recording a Paid Expense (qbExpense)

Use when: Purchase already paid via credit card, check, debit, or cash.

1. Look up vendor via qbMasterData
2. Look up expense account AND payment account (bank/credit card account)
3. Check for duplicates
4. Propose with: vendor, date, payment method, account, line items
5. On confirmation, call qbExpense with:
   - vendorId
   - accountId (payment account — the bank or credit card used)
   - txnDate
   - paymentType ("Check", "CreditCard", "Cash")
   - lines: array of { amount, accountId (expense account), description }
   - Optional: checkNumber (if check), referenceNumber, memo

### Paying a Bill (qbBillPayment)

Use when: Paying an existing unpaid bill.

1. Look up unpaid bills: qbFetchTransactions for vendor's open bills
2. Identify which bill(s) to pay — confirm with user if multiple
3. Look up payment account (bank account)
4. Propose: bill reference, amount, payment account, date
5. On confirmation, call qbBillPayment with:
   - vendorId
   - bankAccountId (account paying from)
   - txnDate
   - lines: array of { billId, amount } — linking to specific bills
   - paymentType ("Check" or "CreditCard")

## Accounts Receivable (AR) Workflows

### Creating a Customer Invoice (qbInvoice)

Use when: Billing a customer for goods/services not yet paid.

1. Look up customer via qbMasterData (type: "Customer")
2. Look up items or service accounts
3. Check for duplicate invoices
4. Propose: customer, date, due date, line items, total
5. On confirmation, call qbInvoice with:
   - customerId
   - txnDate
   - dueDate
   - lines: array of { amount, itemId or accountId, description, quantity, rate }
   - Optional: invoiceNumber, memo, class, salesTermId

### Recording a Cash Sale (qbSalesReceipt)

Use when: Customer paid at time of sale (cash, card, immediate payment).

1. Look up customer (or use generic if walk-in)
2. Look up items/accounts
3. Look up deposit account (where money goes)
4. Propose: customer, date, items, payment method, total
5. On confirmation, call qbSalesReceipt with:
   - customerId
   - depositToAccountId
   - txnDate
   - paymentMethodId
   - lines: array of { amount, itemId or accountId, description }

### Receiving Payment on Invoice (qbReceivePayment)

Use when: Customer is paying an existing outstanding invoice.

1. Look up customer's open invoices: qbFetchTransactions
2. Identify which invoice(s) being paid — confirm with user
3. Look up deposit account
4. Propose: customer, invoice reference, amount, deposit account
5. On confirmation, call qbReceivePayment with:
   - customerId
   - depositToAccountId (or "Undeposited Funds")
   - txnDate
   - totalAmount
   - lines: array of { invoiceId, amount } — linking to specific invoices

### Recording a Bank Deposit (qbDeposit)

Use when: Depositing received payments or other funds into bank.

1. Look up bank account via qbMasterData
2. Determine what's being deposited (received payments, cash, other)
3. Propose: bank account, date, deposit items, total
4. On confirmation, call qbDeposit with:
   - depositToAccountId
   - txnDate
   - lines: array of { amount, accountId, description, entityId (optional) }

### Issuing a Customer Refund (qbRefundReceipt)

Use when: Refunding a customer for a returned item or cancelled service.

1. Look up original transaction if applicable
2. Look up customer and refund account
3. Propose: customer, amount, reason, refund method
4. On confirmation, call qbRefundReceipt with:
   - customerId
   - depositToAccountId (account refund comes from)
   - txnDate
   - lines: array of { amount, itemId or accountId, description }

## General Ledger Workflows

### Journal Entry (qbJournalEntry)

Use when: Adjusting entries, reclassifications, or corrections that don't
fit other transaction types.

1. Identify accounts to debit and credit
2. Verify accounts via qbMasterData
3. Ensure debits = credits
4. Propose: date, debit lines, credit lines, memo
5. On confirmation, call qbJournalEntry with:
   - txnDate
   - lines: array of { accountId, amount, type ("Debit" or "Credit"), description }
   - Optional: adjustmentType, memo

### Voiding a Transaction (qbVoidTransaction)

Use when: Cancelling a previously recorded transaction.

1. Look up the transaction: qbFetchTransactions to get ID and SyncToken
2. Confirm the exact transaction with user (show details)
3. Explain that voiding is permanent and cannot be undone
4. On explicit confirmation, call qbVoidTransaction with:
   - transactionType (Bill, Invoice, Expense/Purchase, etc.)
   - transactionId
   - syncToken

## Edge Cases

### Transaction Date Required
Always require a date before creating any transaction. If user doesn't
provide one, ask. Never default to today without confirming.

### Multi-Line Transactions
Some transactions have multiple line items (e.g., bill with several expense
categories). Ensure each line has its own account and amount.

### Undeposited Funds
When receiving payments, default to "Undeposited Funds" if the user hasn't
specified a deposit account. The deposit step moves funds to the actual bank.

### Sales Tax
If applicable, look up tax rates via qbMasterData (type: "TaxCode") and
include in invoice/sales receipt lines.
