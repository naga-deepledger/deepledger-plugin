---
name: QuickBooks Bookkeeping
description: >
  This skill should be used when the user asks to "record an expense",
  "create an invoice", "pay a bill", "record a bill", "make a payment",
  "record a sale", "create a journal entry", "void a transaction",
  "deposit funds", "issue a refund", "create a vendor", "add a customer",
  "categorize a transaction", "check for duplicates", or any QuickBooks
  transaction recording task. Also activates proactively when processing
  documents, bank feeds, AI tasks, or approved review queue items. Operates
  autonomously — records transactions directly when document-backed and
  confident, escalates only when information is missing or accuracy is at risk.
version: 3.0.0
---

## Purpose

Provide autonomous bookkeeping expertise for recording QuickBooks Online
transactions via DeepLedger MCP tools. Minimize user friction — infer
intelligently, act decisively, ask only when ambiguity would cause
incorrect accounting.

## Operating Principle

**Execute autonomously with document-backed confidence.** The goal is clean,
audit-ready books maintained proactively. Every transaction recorded must be
backed by a source document and verified against duplicates. Every number
impacts financials, tax returns, and disclosures — accuracy is non-negotiable.

## Operating Framework

Analyze → Verify → Execute → Log

Every write operation follows this cycle. Verification means: (1) source
document exists, (2) master data confirmed via qbMasterData, (3) no
duplicates via qbFetchTransactions, (4) confidence score ≥ 80%. Read
operations execute autonomously without any checks.

## Document-Backing Rule (CRITICAL)

Every transaction MUST be backed by a source document before recording.
Acceptable sources:
- **Documents table** — fetchDocuments() for uploaded receipts, invoices, statements
- **Bank feed** — qbCategorizeBankFeed for Teller bank transactions
- **Local files** — receipts or invoices from local folders
- **Email/Gmail** — invoice attachments, payment confirmations
- **Google Drive / Shared Drive** — shared documents, statements
- **CSV/Excel uploads** — bulk transaction imports
- **User-provided description** — sufficient detail (vendor, amount, date, purpose)

If no source document exists, skip this transaction.
Never record undocumented transactions.

## When to Execute vs. When to Escalate

### Execute AUTONOMOUSLY (always — don't ask):
- **Transaction type** — determine from context ("paid" = expense,
  "got a bill" = bill, "invoiced" = invoice)
- **Date** — use the date given; if none given, use today
- **Account/category** — if vendor has history, follow the pattern silently;
  if vendor is new but category is obvious (e.g., "Uber" → Travel), use it
- **Single vendor/customer match** — if only one result or one obvious
  fuzzy match, use it directly
- **No duplicates found** — proceed silently, don't report "no duplicates"
- **Payment method** — infer from context or use reasonable default
- **Memo** — auto-generate from the transaction description
- **Document attachment** — attach source doc to QB transaction after recording

### Flag for review ONLY when (inaccuracy risk):
- **No source document** — request the supporting document
- **Multiple vendor/customer matches** — present options, ask which one
- **No category history AND genuinely ambiguous** — show top options, ask
- **Missing amount** — cannot proceed without this
- **Purchases over $5,000** — "Fixed asset or expense?"
- **Potential duplicate detected** — show the match, ask if this is new
- **Multiple outstanding bills/invoices** — ask which to apply payment to
- **Confidence < 60%** — genuinely uncertain, need human input

## Transaction Recording Process

### Step 1: Identify Source Document
1. Check documents table via fetchDocuments() for matching uploads
2. Check bank feed via qbCategorizeBankFeed for matching bank transactions
3. If from user description: verify sufficient detail (vendor, amount, date, purpose)
4. If no source found: skip this transaction

### Step 2: Infer and Fetch
1. Parse the source document or description to determine type, amount, entity, date, category
2. Fetch master data via qbMasterData — vendors, customers, accounts
3. If entity has a single match, select it. If fuzzy match is obvious, select it
4. Fetch entity's recent transactions to determine categorization pattern

### Step 3: Verify (autonomous)
1. Query via qbFetchTransactions for same entity, date range (±3 days), similar amount
2. If NO matches → proceed (do not mention duplicate check)
3. If potential match found → flag for review, ask if this is a new transaction
4. Compute confidence score (base 50% + vendor history + amount range + description clarity)
5. Confidence ≥ 80%: proceed to execute
6. Confidence 60-79%: execute but flag for review queue
7. Confidence < 60%: flag for review

### Step 4: Execute
- Execute using the appropriate tool immediately
- Attach source document to QB transaction via qbGetUploadUrl
- Update agentMemory with vendor→category pattern
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
| Bank transfer / credit card payment | qbTransfer |
| Estimate / quote | qbEstimate |
| Purchase order | qbPurchaseOrder |
| Recurring transaction (list/create/delete) | qbRecurringTransaction |
| Budget (list/create/read/delete) | qbBudget |
| Vendor credit / customer credit memo | qbCredit |
| Void/delete transaction | qbVoidTransaction |
| Attach receipt/document | qbGetUploadUrl |

### Browser-Only Features (no API)

These features require browser automation via Chrome MCP and cannot be
done through the QBO API:

| Feature | Command | API Status |
|---------|---------|-----------|
| Bank reconciliation | `/reconcile` | No API. Can query reconciliation status per transaction but cannot perform reconciliation |
| Recurring txn edit | `/recurring` | **API supports list/read/create/delete** via qbRecurringTransaction. Only update/edit requires browser |
| Bank rules | `/bank-rules` | No API |
| Bank feed matching | (via `/reconcile`) | No API. Bank feed items ("For Review") not accessible |
| Budget edit | (via QBO UI) | **API supports list/read/create/delete** via qbBudget. Only update/edit requires browser |
| Audit log | (via QBO UI) | No API |
| 1099 filing | (via QBO UI) | Vendor1099 flag settable via API; form generation/filing is UI-only |

## Smart Defaults

| Field | Default | Override |
|-------|---------|---------|
| Date | Today (if not specified and transaction is recent) | User specifies a date |
| Payment method | Infer from context or use reasonable default | User specifies method explicitly |
| Deposit account | Undeposited Funds (for received payments) | User specifies account |
| Memo | Auto-generated from description | User provides memo |
| Category | Step 1: vendor history → Step 2: description + vendor name → Step 3: ask | User requests different account |

## Capitalization Threshold

For purchases over $5,000 (or organization's threshold), flag for review:
"This purchase is over [threshold] — should this be recorded
as a fixed asset or expensed?"

Under $5,000: always expense autonomously.

## Before Recording Payments

Check for outstanding invoices/bills first via qbFetchTransactions.
- Exactly one match → link automatically
- Multiple matches → flag for review asking which one(s) to apply
- No matches → proceed as standalone payment

## Batch Processing

When multiple transactions are provided at once:
1. Process all of them autonomously, verifying each against source documents
2. Execute each transaction that passes verification (document-backed, duplicate-checked, confident)
3. Flag uncertain ones for review queue
4. Present a summary report of all actions taken (recorded, flagged, escalated)

## Optional Fields

Include class, billable status, check number, and reference number
when available from the source document. These strengthen the audit trail.

## Document Attachment (REQUIRED)

After every successful transaction recording:
1. If source is from documents table → attach via qbGetUploadUrl
2. If source is bank feed → document is auto-linked via Teller
3. If source is user description → note in memo field as source reference

## Additional Resources

### Reference Files

- **`references/transaction-workflows.md`** — Step-by-step recording
  procedures for each transaction type with edge cases
- **`references/duplicate-detection.md`** — Detailed duplicate checking
  methodology and fuzzy matching guidance
- **`references/categorization.md`** — Account categorization rules,
  common mappings, and consistency patterns
- **`references/auto-categorization-model.md`** — Confidence scoring
  model, learning loop from Review Queue, AI reasoning format, and
  decision thresholds for auto-apply vs. flag-for-review
- **`references/recurring-transactions.md`** — Recurring transaction
  detection, pattern matching, and missing transaction alerts
- **`references/sales-tax.md`** — Sales tax handling for invoices,
  receipts, and purchases; tax-exempt scenarios
- **`references/error-recovery.md`** — API error handling, edge cases
  (negative amounts, partial payments, backdated transactions, etc.)
- **`references/qbo-api-capabilities.md`** — Complete matrix of what's
  available via API vs. browser-only, MCP tool mapping, rate limits
