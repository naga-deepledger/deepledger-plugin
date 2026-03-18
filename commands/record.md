---
description: Record a transaction (expense, bill, invoice, etc.)
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: <description of transaction>
---

Record a transaction based on the user's description: "$ARGUMENTS"

Operate autonomously — infer everything you can from the description,
vendor history, and smart defaults. Only ask when ambiguity would cause
incorrect accounting.

Steps:
1. Parse the description to determine (infer ALL of these, don't ask):
   - Transaction type — "paid" = expense, "bill from" = bill, "invoice" = invoice, etc.
   - Amount
   - Vendor or customer name
   - Date — use what's given, default to today if not specified
   - Account/category — check vendor history first, then infer from name/description
2. Use qbMasterData to look up relevant vendors/customers/accounts
   - Single match or obvious fuzzy match → use it, don't confirm
   - Multiple ambiguous matches → list them and ask which one
   - Not found → ask if they want to create it
3. Check vendor's recent transactions to determine categorization pattern
   - Consistent history → use that account silently
   - No history but obvious category → infer from vendor name
   - Genuinely ambiguous → ask with top options
4. Check for duplicates via qbFetchTransactions (silently):
   - No matches → proceed without mentioning duplicate check
   - Potential match found → show it, ask if this is new
5. Propose the transaction with full details:
   - Transaction type
   - Vendor/customer name and ID
   - Account name, AcctNum (4-6 digit code), and Account ID
   - Line items with descriptions and amounts
   - Total amount and date
   - Mark inferred fields: "(based on history)" or "(inferred)"
6. Wait for user confirmation — present a clean, ready-to-approve proposal
7. Create using the appropriate tool:
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
8. Confirm success with transaction ID and summary

Smart defaults (apply without asking):
- Date not specified → today
- Payment method not specified → CreditCard for <$500, Check for ≥$500
- Memo → auto-generate from transaction description
- Under $5,000 → always expense (don't ask about capitalization)

Only ask about:
- Purchases over $5,000: "Fixed asset or expense?"
- Outstanding invoices/bills when recording a payment (if multiple exist)
- Missing amount (cannot proceed without it)
