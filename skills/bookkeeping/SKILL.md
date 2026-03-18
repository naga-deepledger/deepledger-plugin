---
name: QuickBooks Bookkeeping
description: >
  This skill should be used when the user asks to "record an expense",
  "create an invoice", "pay a bill", "record a bill", "make a payment",
  "record a sale", "create a journal entry", "void a transaction",
  "deposit funds", "issue a refund", "create a vendor", "add a customer",
  "categorize a transaction", "check for duplicates", or any QuickBooks
  transaction recording task.
version: 2.0.0
---

## Purpose

Provide autonomous bookkeeping expertise for recording QuickBooks Online
transactions via DeepLedger MCP tools. Minimize user friction — infer
intelligently, act decisively, ask only when ambiguity would cause
incorrect accounting.

## Operating Principle

**Act first, ask only when necessary.** The goal is clean, accurate
accounting with minimal back-and-forth. Users should be able to say
"I paid $200 to Staples for office supplies" and get a ready-to-confirm
proposal without any intermediate questions.

## Operating Framework

Analyze → Propose → Confirm → Execute

Every write operation follows this cycle. Read operations (fetching data,
reports, queries) execute autonomously without confirmation.

## When to Ask vs. When to Infer

### NEVER ask about (always infer):
- **Transaction type** — determine from context ("paid" = expense,
  "got a bill" = bill, "invoiced" = invoice)
- **Date** — use the date given; if none given, use today
- **Account/category** — if vendor has history, follow the pattern silently;
  if vendor is new but category is obvious (e.g., "Uber" → Travel), use it
- **Single vendor/customer match** — if only one result or one obvious
  fuzzy match, use it without confirming
- **No duplicates found** — proceed silently, don't report "no duplicates"
- **Payment method** — infer from context or use reasonable default
- **Memo** — auto-generate from the transaction description

### ALWAYS ask about (ambiguity causes bad accounting):
- **Multiple vendor/customer matches** — present options, ask which one
- **No category history AND genuinely ambiguous** — show top options, ask
- **Missing amount** — cannot proceed without this
- **Purchases over $5,000** — "Fixed asset or expense?"
- **Potential duplicate detected** — show the match, ask if this is new
- **Multiple outstanding bills/invoices** — ask which to apply payment to

## Transaction Recording Process

### Step 1: Infer and Fetch (autonomous)
1. Parse the user's description to determine type, amount, entity, date, category
2. Fetch master data via qbMasterData — vendors, customers, accounts
3. If entity has a single match, select it. If fuzzy match is obvious, select it
4. Fetch entity's recent transactions to determine categorization pattern

### Step 2: Check Duplicates (autonomous)
1. Query via qbFetchTransactions for same entity, date range (±3 days), similar amount
2. If NO matches → proceed silently (do not mention duplicate check to user)
3. If potential match found → show it and ask if this is a new transaction

### Step 3: Propose (present complete, ready to confirm)
Display a clean proposal with:
- Transaction type
- Vendor/customer name and ID
- Account name, AcctNum (4-6 digit code), and Account ID
- Line items with descriptions and amounts
- Total amount and date
- Any auto-inferred fields marked with source: "(based on history)" or "(inferred)"

### Step 4: Confirm and Execute
- Wait for user confirmation ("yes", "confirm", "go ahead", etc.)
- Execute using the appropriate tool
- Report success with transaction ID

## Tool Selection Guide

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

## Smart Defaults

| Field | Default | Override |
|-------|---------|---------|
| Date | Today (if not specified and transaction is recent) | User specifies a date |
| Payment method | CreditCard for <$500, Check for ≥$500 | User specifies method |
| Deposit account | Undeposited Funds (for received payments) | User specifies account |
| Memo | Auto-generated from description | User provides memo |
| Category | Vendor's most recent pattern | User requests different account |

## Capitalization Threshold

For purchases over $5,000 (or organization's threshold), ask:
"This is over [threshold] — should this be recorded as a fixed asset or expensed?"

Under $5,000: always expense without asking.

## Before Recording Payments

Check for outstanding invoices/bills first via qbFetchTransactions.
- Exactly one match → link automatically, mention it in proposal
- Multiple matches → ask which one(s) to apply
- No matches → proceed as standalone payment

## Batch Processing

When multiple transactions are provided at once:
1. Process all of them, inferring details for each
2. Present a summary table of all proposed transactions
3. Get a single bulk confirmation
4. Execute all and report results

## Optional Fields

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
