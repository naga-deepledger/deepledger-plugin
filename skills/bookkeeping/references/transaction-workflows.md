# Transaction Workflows — Autonomous Recording Procedures

## General Principles

1. **Infer first, ask second.** Use context, vendor history, and smart
   defaults to fill in every field possible before presenting to the user.
2. **Date defaults to today** if not specified and the transaction is recent.
   Only ask for date when the user describes something clearly in the past
   without specifying when (e.g., "I got a bill from someone last week").
3. **Silent duplicate checks.** Check for duplicates on every transaction
   but only surface results when a potential match is found.
4. **Single-match auto-select.** When a vendor/customer lookup returns one
   result or one obvious fuzzy match, use it without confirming.

## Accounts Payable (AP) Workflows

### Recording a Vendor Bill (qbBill)

Use when: Received a bill/invoice from a vendor that has NOT been paid yet.
Trigger words: "got a bill", "received an invoice", "owes", "unpaid bill"

1. Look up vendor via qbMasterData (type: "Vendor", search by name)
   - Single match → use it
   - Multiple similar → ask which one
   - Not found → ask user if they want to create it
2. Look up expense account — check vendor's recent history first, then
   infer from vendor name/description, then ask only if ambiguous
3. Check for duplicates silently — only surface if match found
4. Propose bill with: vendor, date (default today), due date (if known),
   line items, accounts, total
5. On confirmation, call qbBill with:
   - vendorId (from master data lookup)
   - txnDate (bill date — default today if not specified)
   - dueDate (if known, otherwise omit)
   - lines: array of { amount, accountId, description }
   - Optional: referenceNumber, memo (auto-generate from description), class

### Recording a Paid Expense (qbExpense)

Use when: Purchase already paid via credit card, check, debit, or cash.
Trigger words: "paid", "bought", "purchased", "charged", "spent"

1. Look up vendor via qbMasterData — auto-select single/obvious match
2. Look up expense account (from vendor history) AND payment account
   - Payment account: infer from context ("credit card" → find CC account,
     "check" → find checking account). If not specified, default to the
     company's primary checking or credit card account
3. Check for duplicates silently
4. Propose with: vendor, date (default today), payment method (infer),
   account, line items
5. On confirmation, call qbExpense with:
   - vendorId
   - accountId (payment account — the bank or credit card used)
   - txnDate (default today)
   - paymentType (only if user specified: "by card" → "CreditCard",
     "by check" → "Check", "cash" → "Cash"; otherwise leave for user
     to confirm in proposal)
   - lines: array of { amount, accountId (expense account), description }
   - Optional: checkNumber (if check), referenceNumber, memo (auto-generate)

### Paying a Bill (qbBillPayment)

Use when: Paying an existing unpaid bill.
Trigger words: "pay the bill", "pay [vendor]", "settle the bill"

1. Look up unpaid bills: qbFetchTransactions for vendor's open bills
   - Single open bill → auto-select it
   - Multiple open bills → ask which one(s)
   - No open bills → inform user, suggest qbExpense instead
2. Look up payment account (bank account) — use default if not specified
3. Propose: bill reference, amount, payment account, date (default today)
4. On confirmation, call qbBillPayment with:
   - vendorId
   - bankAccountId (account paying from)
   - txnDate (default today)
   - lines: array of { billId, amount } — linking to specific bills
   - paymentType ("Check" or "CreditCard" — infer from context)

## Accounts Receivable (AR) Workflows

### Creating a Customer Invoice (qbInvoice)

Use when: Billing a customer for goods/services not yet paid.
Trigger words: "invoice", "bill [customer]", "charge [customer]"

1. Look up customer via qbMasterData — auto-select single/obvious match
2. Look up items or service accounts — use history if available
3. Check for duplicate invoices silently
4. Propose: customer, date (default today), due date, line items, total
5. On confirmation, call qbInvoice with:
   - customerId
   - txnDate (default today)
   - dueDate (default Net 30 from txnDate if not specified)
   - lines: array of { amount, itemId or accountId, description, quantity, rate }
   - Optional: invoiceNumber, memo (auto-generate), class, salesTermId

### Recording a Cash Sale (qbSalesReceipt)

Use when: Customer paid at time of sale (cash, card, immediate payment).
Trigger words: "sold", "cash sale", "walk-in sale", "paid at point of sale"

1. Look up customer (or use generic if walk-in/not specified)
2. Look up items/accounts — use history or infer
3. Look up deposit account — use default if not specified
4. Propose: customer, date (default today), items, payment method, total
5. On confirmation, call qbSalesReceipt with:
   - customerId
   - depositToAccountId (default to primary checking)
   - txnDate (default today)
   - paymentMethodId (infer from context)
   - lines: array of { amount, itemId or accountId, description }

### Receiving Payment on Invoice (qbReceivePayment)

Use when: Customer is paying an existing outstanding invoice.
Trigger words: "[customer] paid", "received payment", "payment on invoice"

1. Look up customer's open invoices: qbFetchTransactions
   - Single open invoice → auto-link
   - Multiple open invoices → ask which one(s)
   - No open invoices → inform user, suggest qbSalesReceipt or qbDeposit
2. Look up deposit account — default to "Undeposited Funds" if not specified
3. Propose: customer, invoice reference, amount, deposit account
4. On confirmation, call qbReceivePayment with:
   - customerId
   - depositToAccountId (or "Undeposited Funds")
   - txnDate (default today)
   - totalAmount
   - lines: array of { invoiceId, amount } — linking to specific invoices

### Recording a Bank Deposit (qbDeposit)

Use when: Depositing received payments or other funds into bank.
Trigger words: "deposit", "deposited", "bank deposit"

1. Look up bank account via qbMasterData — use primary if not specified
2. Determine what's being deposited (received payments, cash, other)
3. Propose: bank account, date (default today), deposit items, total
4. On confirmation, call qbDeposit with:
   - depositToAccountId
   - txnDate (default today)
   - lines: array of { amount, accountId, description, entityId (optional) }

### Issuing a Customer Refund (qbRefundReceipt)

Use when: Refunding a customer for a returned item or cancelled service.
Trigger words: "refund", "return", "give back"

1. Look up original transaction if applicable
2. Look up customer and refund account — auto-select if obvious
3. Propose: customer, amount, reason, refund method
4. On confirmation, call qbRefundReceipt with:
   - customerId
   - depositToAccountId (account refund comes from)
   - txnDate (default today)
   - lines: array of { amount, itemId or accountId, description }

### Issuing a Vendor Credit or Customer Credit Memo (qbCredit)

Use when: Vendor issued a credit for returned goods, billing error, or
overpayment; OR issuing a credit memo to a customer.
Trigger words: "credit", "credit memo", "vendor credit", "credit note"

1. Determine if this is a vendor credit or customer credit memo — infer
   from context (mention of vendor → vendor credit, customer → credit memo)
2. Look up vendor or customer via qbMasterData — auto-select single match
3. Look up the relevant accounts — use history or infer
4. Check for duplicates silently
5. If linked to a specific bill or invoice, fetch it via qbFetchTransactions
   - Single matching bill/invoice → auto-link
   - Multiple → ask which one
6. Propose: entity, date (default today), line items, accounts, total credit
7. On confirmation, call qbCredit with:
   - vendorId or customerId
   - txnDate (default today)
   - lines: array of { amount, accountId, description }
   - Optional: memo, referenceNumber, linkedTxnId (bill or invoice to apply against)

**Note:** Vendor credits reduce what you owe; customer credit memos reduce
what the customer owes. Ensure the user understands the direction of the credit.

## General Ledger Workflows

### Journal Entry (qbJournalEntry)

Use when: Adjusting entries, reclassifications, or corrections that don't
fit other transaction types.
Trigger words: "adjust", "reclassify", "journal entry", "correct"

1. Identify accounts to debit and credit — infer from description
2. Verify accounts via qbMasterData
3. Ensure debits = credits
4. Propose: date (default today), debit lines, credit lines, memo (auto-generate)
5. On confirmation, call qbJournalEntry with:
   - txnDate (default today)
   - lines: array of { accountId, amount, type ("Debit" or "Credit"), description }
   - Optional: adjustmentType, memo

### Voiding a Transaction (qbVoidTransaction)

Use when: Cancelling a previously recorded transaction.
Trigger words: "void", "cancel", "delete", "undo"

1. Look up the transaction: qbFetchTransactions to get ID and SyncToken
   - If user provides enough detail to identify uniquely → auto-find
   - If ambiguous → ask for clarification
2. Show transaction details to user for confirmation
3. Clearly state that voiding is permanent and cannot be undone
4. On explicit confirmation, call qbVoidTransaction with:
   - transactionType (Bill, Invoice, Expense/Purchase, etc.)
   - transactionId
   - syncToken

### Bank Transfer (qbJournalEntry — Transfer Pattern)

Use when: Moving money between the company's own bank accounts, or between
a bank account and a credit card (e.g., credit card payment).
Trigger words: "transfer", "move money", "transfer from savings", "pay credit card",
"moved funds", "wire between accounts"

**Note:** The QBO API has a native Transfer entity (FromAccountRef, ToAccountRef,
Amount). If the MCP server exposes a qbTransfer tool, prefer that. Otherwise,
use a Journal Entry with a specific debit/credit pattern between bank accounts.

1. Identify source and destination accounts:
   - Look up both accounts via qbMasterData (type: "Account", accountType: "Bank" or "CreditCard")
   - "Transfer from savings to checking" → debit Checking, credit Savings
   - "Pay off credit card from checking" → debit Credit Card, credit Checking
   - If only one account mentioned, ask which is the other
2. Validate both accounts exist and are asset/liability accounts (Bank, CreditCard, OtherCurrentAsset)
3. Check for duplicates silently (same amount, same date, same accounts)
4. Propose: source account, destination account, amount, date (default today), memo
5. On confirmation, call qbJournalEntry with:
   - txnDate (default today)
   - lines:
     - { accountId: destinationAccountId, amount: transferAmount, type: "Debit", description: "Transfer from [source]" }
     - { accountId: sourceAccountId, amount: transferAmount, type: "Credit", description: "Transfer to [destination]" }
   - memo: "Transfer: [source] → [destination]"

**Important distinctions:**
- Transfer ≠ expense. Moving money between your own accounts is NOT an expense.
  Never record a credit card payment as an expense — it's a transfer.
- Transfer ≠ payment. Paying a vendor is a bill payment or expense, not a transfer.
  Only use this workflow when both sides are the company's own accounts.
- Credit card payments: Debit the Credit Card account (reduces liability),
  Credit the Bank account (reduces cash). This is a transfer, not an expense.

### Estimates and Quotes

Use when: Creating a preliminary price proposal for a customer before invoicing.
Trigger words: "estimate", "quote", "proposal", "bid", "price quote"

The QBO API fully supports Estimates (full CRUD). If the MCP server exposes
a qbEstimate tool, use it with:
- customerId
- txnDate (default today)
- expirationDate (default 30 days from txnDate)
- lines: array of { amount, itemId or accountId, description, quantity, rate }
- Optional: estimateNumber, memo, class, salesTermId

If no qbEstimate MCP tool is available, offer alternatives:
- Create the estimate in QBO UI
- Create a draft invoice marked "ESTIMATE — DO NOT PAY" in the memo

**Conversion:** When accepted, convert to invoice (copy line items).

### Purchase Orders

Use when: Creating a purchase commitment to a vendor before receiving goods/bill.
Trigger words: "purchase order", "PO", "order from vendor"

The QBO API fully supports Purchase Orders (full CRUD). If the MCP server
exposes a qbPurchaseOrder tool, use it with:
- vendorId
- txnDate (default today)
- lines: array of { amount, itemId or accountId, description, quantity, rate }
- Optional: PONumber, memo, shipAddr, expectedDate

If no qbPurchaseOrder MCP tool is available, offer alternatives:
- Create the PO in QBO UI
- Record the bill (qbBill) when goods arrive instead
- Track commitment via memo or attached document (qbGetUploadUrl)

**Receiving:** When goods arrive, convert PO to bill (copy line items to qbBill).

## Edge Cases

### Transaction Date
- If the user specifies a date → use it exactly
- If the user says "today" or implies recency → use today's date
- If the user describes a past event without a date (e.g., "last week") →
  ask for the specific date (accounting requires exact dates)
- NEVER silently default to today for something clearly in the past

### Multi-Line Transactions
Some transactions have multiple line items (e.g., bill with several expense
categories). Parse each item from the description, assign accounts to each,
and present as a multi-line proposal.

### Undeposited Funds
When receiving payments, default to "Undeposited Funds" if the user hasn't
specified a deposit account. The deposit step moves funds to the actual bank.

### Sales Tax
If applicable, look up tax rates via qbMasterData (type: "TaxCode") and
include in invoice/sales receipt lines.
