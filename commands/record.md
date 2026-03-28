---
description: Record a transaction (expense, bill, invoice, etc.)
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: <description of transaction>
---

Record a transaction based on the user's description: "$ARGUMENTS"

Operate fully autonomously — execute directly when document-backed and
confident. Every number impacts financials, tax returns, and disclosures.
Flag for review only when information is missing or accuracy is at risk.

Steps:
1. **Identify source document** — check documents table (fetchDocuments),
   bank feed (qbCategorizeBankFeed), or verify user description has
   sufficient detail (vendor, amount, date, purpose). If no source: skip this transaction.
2. **Parse and infer** (infer ALL of these, don't ask):
   - Transaction type — "paid" = expense, "bill from" = bill, "invoice" = invoice, etc.
   - Amount
   - Vendor or customer name
   - Date — use what's given, default to today if not specified
   - Account/category — check vendor history first, then infer from name/description
3. **Verify master data** via qbMasterData:
   - Single match or obvious fuzzy match → use it directly
   - Multiple ambiguous matches → flag for review
   - Not found → flag for review asking if should create it
4. **Check categorization pattern** from vendor's recent transactions:
   - Consistent history → use that account
   - No history but obvious category → infer from vendor name
   - Genuinely ambiguous → flag for review with top options
5. **Check duplicates** via qbFetchTransactions:
   - No matches → proceed
   - Potential match found → flag for review, ask if new
6. **Execute immediately** using the appropriate tool:
   - Paid expense (card/check/cash) → qbExpense
   - Vendor bill (unpaid) → qbBill
   - Bill payment → qbBillPayment
   - Customer invoice → qbInvoice
   - Receive payment → qbReceivePayment
   - Cash sale → qbSalesReceipt
   - Bank deposit → qbDeposit
   - Refund → qbRefundReceipt
   - Vendor credit or customer credit memo → qbCredit
   - Bank transfer or credit card payment → qbTransfer
   - Adjusting entry → qbJournalEntry
7. **Attach source document** to QB transaction via qbGetUploadUrl
8. **Learn** — update agentMemory for patterns
9. Report success with transaction ID and summary

Smart defaults (apply autonomously):
- Date not specified → today
- Payment method → infer from context or use reasonable default
- Memo → auto-generate from transaction description
- Under $5,000 → always expense (don't ask about capitalization)
- Category → vendor history first, then description keywords, then
  single-purpose vendor name; escalate ONLY for multi-purpose vendors
  (Amazon, Costco, etc.) when description is vague

Only flag for review for:
- No source document (request it)
- Purchases over $5,000: "Fixed asset or expense?"
- Outstanding invoices/bills when recording a payment (if multiple exist)
- Missing amount (cannot proceed without it)
- Potential duplicates detected
