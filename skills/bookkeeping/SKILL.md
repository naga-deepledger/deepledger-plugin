---
name: QuickBooks Bookkeeping
description: >
  This skill should be used when the user asks to "record an expense",
  "create an invoice", "pay a bill", "record a bill", "make a payment",
  "record a sale", "create a journal entry", "void a transaction",
  "deposit funds", "issue a refund", "create a vendor", "add a customer",
  "categorize a transaction", "check for duplicates", or any QuickBooks
  transaction recording task.
version: 1.0.0
---

## Purpose

Provide bookkeeping expertise for recording QuickBooks Online transactions
via DeepLedger MCP tools.

## Operating Framework

Analyze → Propose → Confirm → Execute

Every write operation follows this cycle. Read operations (fetching data,
reports, queries) execute autonomously without confirmation.

## Transaction Recording Process

### Before Any Write Operation

1. Fetch master data via qbMasterData to identify correct accounts, vendors,
   customers, items, and classes
2. Check for duplicates via qbFetchTransactions — match on date, amount,
   and vendor/customer before creating
3. If vendor/customer name is ambiguous (multiple similar names), confirm
   the exact entity with the user
4. Never assume account/category — always verify with qbMasterData

### Proposing Transactions

Always display in the proposal:
- Account name and category
- AcctNum (user-facing, usually 4-6 digits)
- Account ID (internal, usually 2 digits)
- Vendor/customer name and ID
- Line items with amounts
- Total amount

### Capitalization Threshold

For purchases over $5,000 (or organization's threshold), ask:
"This is over [threshold] — should this be recorded as a fixed asset or expensed?"

### Tool Selection Guide

| Scenario | Tool |
|----------|------|
| Vendor bill (unpaid) | qbBill |
| Pay an existing bill | qbBillPayment |
| Paid expense (card/check/cash) | qbExpense |
| Customer invoice (unpaid) | qbInvoice |
| Receive payment on invoice | qbReceivePayment |
| Cash/card sale (paid immediately) | qbSalesReceipt |
| Bank deposit | qbDeposit |
| Customer refund | qbRefundReceipt |
| Adjusting entry | qbJournalEntry |
| Vendor credit / customer credit memo | qbCredit |
| Void/delete transaction | qbVoidTransaction |
| Attach receipt/document | qbGetUploadUrl |

### Before Recording Payments

Check for outstanding invoices/bills first via qbFetchTransactions. Link
payments to existing documents instead of creating standalone transactions.

### Optional Fields

Prompt for class, billable status, check number, and reference number
only when relevant to the transaction type. Suggest these for audit
trails but do not require them.

## Additional Resources

### Reference Files

- **`references/transaction-workflows.md`** — Step-by-step recording
  procedures for each transaction type with edge cases
- **`references/duplicate-detection.md`** — Detailed duplicate checking
  methodology and fuzzy matching guidance
- **`references/categorization.md`** — Account categorization rules,
  common mappings, and consistency patterns
