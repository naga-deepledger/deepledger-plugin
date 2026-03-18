---
description: Record a transaction (expense, bill, invoice, etc.)
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: <description of transaction>
---

Record a transaction based on the user's description: "$ARGUMENTS"

Steps:
1. Parse the description to determine:
   - Transaction type (expense, bill, invoice, payment, deposit, etc.)
   - Amount
   - Vendor or customer name
   - Date (or ask if not provided)
   - Account/category (or look up)
2. Use qbMasterData to look up relevant vendors/customers/accounts
   - If vendor/customer not found, ask user if they want to create it
   - If multiple similar names, list them and ask user to confirm which one
3. Check for duplicates via qbFetchTransactions:
   - Search by same date, similar amount (±5%), and same vendor/customer
   - If potential duplicates found, show them and ask user to confirm this is new
4. Propose the transaction with full details:
   - Transaction type
   - Vendor/customer name and ID
   - Account name, AcctNum (4-6 digit user-facing code), and Account ID
   - Line items with descriptions and amounts
   - Total amount
   - Date
5. Wait for explicit user confirmation ("yes", "confirm", "go ahead", etc.)
6. Create using the appropriate tool:
   - Paid expense (card/check/cash) → qbExpense
   - Vendor bill (unpaid) → qbBill
   - Bill payment → qbBillPayment
   - Customer invoice → qbInvoice
   - Receive payment → qbReceivePayment
   - Cash sale → qbSalesReceipt
   - Bank deposit → qbDeposit
   - Refund → qbRefundReceipt
   - Vendor credit or customer credit memo → qbCredit
   - Adjusting entry → qbJournalEntry
7. Confirm success with transaction ID and summary

Special rules:
- For purchases over $5,000: ask "Should this be recorded as a fixed asset or expensed?"
- For payments: check for outstanding invoices/bills to link against first
- Suggest class, memo, and reference number for audit trail but don't require them
