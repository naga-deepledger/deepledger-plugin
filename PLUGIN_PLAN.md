# DeepLedger Claude Code Plugin — Implementation Plan

> **Status**: Draft for review — implementation pending
> **Date**: 2026-03-14
> **Type**: SaaS plugin (remote MCP server, no local process)
> **IMPORTANT**: This document is fully self-contained. It has the complete
> content for every file. No external context or access to any other repo
> is needed to build this plugin.

---

## 1. What This Plugin Is

A Claude Code plugin that connects to the **hosted DeepLedger MCP server**
(already running at `https://agent.deepledger.ai`) and layers bookkeeping
intelligence on top. Users install the plugin, authenticate via OAuth, and
get an AI bookkeeper with zero configuration.

**The plugin contains no server code.** It is a collection of markdown files
and JSON configs that tells Claude Code:
1. Where to connect (remote MCP server via SSE)
2. How to authenticate (OAuth 2.1 — handled automatically)
3. How to use the tools intelligently (skills, agents, commands, hooks)

### What the Remote Server Provides (for context, NOT part of this plugin)

The DeepLedger MCP server exposes these QuickBooks tools:
- **qbMasterData** — Retrieve, create, or update master data (vendors, customers, accounts, items, classes, tax rates). Use operation='retrieve' (default) for lookups, 'create'/'update' for writes.
- **qbFetchTransactions** — Look up specific transactions for editing, voiding, paying, linking, or duplicate-checking (returns IDs/SyncTokens/line items)
- **qbReports** — Generate financial reports (ProfitAndLoss, BalanceSheet, CashFlow, AgedReceivables, AgedPayables, TrialBalance, TransactionList, CustomerIncome, VendorExpenses). TransactionList returns individual transactions with filtering by type, grouping by Customer/Vendor/Account, and client-side account ID filtering.
- **qbBill** — Create vendor bills (unpaid purchases)
- **qbBillPayment** — Record bill payments
- **qbExpense** — Record paid expenses (check/card/cash)
- **qbInvoice** — Create customer invoices (unpaid sales)
- **qbReceivePayment** — Record customer payments
- **qbSalesReceipt** — Record paid sales (cash/card)
- **qbDeposit** — Record bank deposits
- **qbRefundReceipt** — Issue customer refunds
- **qbJournalEntry** — Manual accounting adjustments
- **qbCredit** — Create vendor/customer credits
- **qbVoidTransaction** — Void/delete transactions
- **qbGetUploadUrl** — Get pre-signed upload URL for attaching receipts/documents to QB transactions. WORKFLOW: Call this tool → returns curlCommand. Execute the curlCommand to upload. File is NOT attached until curl runs. "Expense" in QB UI = "Purchase" entityType.

The server handles:
- OAuth 2.1 with PKCE, dynamic client registration, token rotation
- Supabase-based multi-tenant auth (user logs in, selects organization)
- All QuickBooks API calls on behalf of the authenticated user's org

---

## 2. Claude Code Plugin Concepts

A Claude Code plugin is a directory with this structure. Claude Code
auto-discovers components from conventional directories:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json          # REQUIRED — Plugin manifest (name, version, etc.)
├── .mcp.json                # MCP server connections (auto-starts on plugin load)
├── commands/                # Slash commands — .md files with YAML frontmatter
├── skills/                  # Auto-activating knowledge — subdirs with SKILL.md
│   └── skill-name/
│       ├── SKILL.md         # REQUIRED per skill — frontmatter + instructions
│       └── references/      # Detailed docs loaded on-demand by Claude
├── agents/                  # Specialized subagents — .md files with frontmatter
└── hooks/
    └── hooks.json           # Event-driven validation/automation
```

### How each component works:

**Commands** (`commands/*.md`): User-invoked via `/command-name`. YAML frontmatter
defines `description`, `allowed-tools`, `argument-hint`, `model`. Body is
instructions for Claude. `$ARGUMENTS` = user's input after the command name.

**Skills** (`skills/*/SKILL.md`): Auto-activate when user's request matches the
`description` field in frontmatter. Body loaded into Claude's context. Reference
files in `references/` loaded on-demand. Write in imperative form, third-person
description.

**Agents** (`agents/*.md`): Specialized subprocesses. Frontmatter needs `name`,
`description` (with `<example>` blocks showing when to trigger), `model`, `color`,
`tools`. Body is second-person system prompt ("You are...").

**Hooks** (`hooks/hooks.json`): JSON config for event handlers. `PreToolUse` hooks
run before tool execution. `matcher` selects which tools. Prompt-based hooks let
Claude reason about context. Return 'approve' or 'block'.

**.mcp.json**: Defines MCP server connections. For remote servers use `type: "sse"`
with a `url`. OAuth is handled automatically by Claude Code when it discovers the
server's `/.well-known/oauth-authorization-server` endpoint.

**Tool naming**: When connected via plugin, tools are prefixed:
`mcp__plugin_<plugin-name>_<server-name>__<tool-name>`
Example: `mcp__plugin_deepledger_deepledger__qbReports`

---

## 3. Directory Structure to Create

```
deepledger-plugin/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json
├── .gitignore
├── README.md
├── commands/
│   ├── pnl.md
│   ├── balance-sheet.md
│   ├── cash-flow.md
│   ├── record.md
│   ├── aging.md
│   └── find.md
├── skills/
│   ├── bookkeeping/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── transaction-workflows.md
│   │       ├── duplicate-detection.md
│   │       └── categorization.md
│   └── financial-analysis/
│       ├── SKILL.md
│       └── references/
│           ├── variance-analysis.md
│           └── warning-signs.md
├── agents/
│   ├── accountant.md
│   └── cfo.md
└── hooks/
    └── hooks.json
```

---

## 4. File Contents — Create Each File Exactly As Shown

### 4.1 `.claude-plugin/plugin.json`

```json
{
  "name": "deepledger",
  "version": "1.0.0",
  "description": "AI bookkeeper with QuickBooks Online integration via DeepLedger",
  "author": {
    "name": "DeepLedger",
    "url": "https://deepledger.ai"
  },
  "homepage": "https://deepledger.ai",
  "repository": "https://github.com/deepledger/deepledger-plugin",
  "license": "MIT",
  "keywords": ["quickbooks", "bookkeeping", "accounting", "finance", "ai-cfo"]
}
```

---

### 4.2 `.mcp.json`

```json
{
  "mcpServers": {
    "deepledger": {
      "type": "sse",
      "url": "https://agent.deepledger.ai/sse"
    }
  }
}
```

---

### 4.3 `.gitignore`

```
.DS_Store
*.swp
*.swo
*~
```

---

### 4.4 `README.md`

```markdown
# DeepLedger Plugin for Claude Code

AI bookkeeper with QuickBooks Online integration. Record transactions,
generate financial reports, and get actionable business insights — all
through natural language.

## Installation

```bash
git clone https://github.com/deepledger/deepledger-plugin.git
claude --plugin-dir ./deepledger-plugin
```

## First Use

1. Start Claude Code with the plugin installed
2. A browser window opens for authentication
3. Log in with your DeepLedger account
4. Select your organization
5. Start using QuickBooks tools via natural language

## Available Commands

| Command | Description |
|---------|-------------|
| `/pnl` | Profit & Loss with variance analysis |
| `/balance-sheet` | Balance sheet snapshot with key ratios |
| `/cash-flow` | Cash flow statement and runway analysis |
| `/record` | Record a transaction |
| `/aging` | AR/AP aging report |
| `/find` | Search transactions |

## Examples

```
"Record a $500 expense to Office Depot for supplies"
"Show me this month's P&L compared to last month"
"Who owes us money?"
"How is my business doing this quarter?"
```

## Requirements

- [Claude Code](https://claude.ai/code) installed
- A DeepLedger account with QuickBooks Online connected
```

---

### 4.5 `commands/pnl.md`

```markdown
---
description: Generate P&L report with variance analysis
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period]
---

Generate a Profit & Loss report. If $ARGUMENTS is provided, use it as
the reporting period (e.g., "this month", "Q1 2026", "last quarter").
If no period specified, default to current month vs prior month.

Steps:
1. Use qbReports with reportType "ProfitAndLoss" for the requested period
2. Use qbReports again for the comparison period (prior month or prior year same period)
3. Calculate variances (% change and $ impact) for each major category:
   - Revenue / Income
   - Cost of Goods Sold
   - Gross Profit (and gross margin %)
   - Operating Expenses (by category)
   - Net Income
4. Highlight significant variances (>10% or >$1,000 change)
5. Translate findings into plain business language — not just "revenue is down 12%"
   but explain what it means and what might be driving it
6. Surface warning signs: margin compression, expense spikes, revenue concentration
7. For any major fluctuation (>$5,000 or >20%), use qbFetchTransactions or
   qbReports with TransactionList to drill into the specific transactions driving it
8. Present in a clean format: summary table, then narrative insights, then recommendations
```

---

### 4.6 `commands/balance-sheet.md`

```markdown
---
description: Balance sheet snapshot with key ratios
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [date]
---

Generate a Balance Sheet report. If $ARGUMENTS is provided, use it as
the as-of date. Default to today.

Steps:
1. Use qbReports with reportType "BalanceSheet" for the requested date
2. Use qbReports again for the comparison date (prior month-end or prior quarter-end)
3. Summarize by major section:
   - Current Assets (cash, AR, inventory)
   - Fixed Assets
   - Current Liabilities (AP, credit cards, short-term debt)
   - Long-term Liabilities
   - Equity
4. Calculate key ratios:
   - Current ratio (current assets / current liabilities)
   - Quick ratio ((current assets - inventory) / current liabilities)
   - Debt-to-equity (total liabilities / total equity)
5. Compare to prior period and highlight significant changes
6. Flag concerns: negative working capital, high debt ratio, declining cash
7. Present: summary table, ratios, period comparison, narrative
```

---

### 4.7 `commands/cash-flow.md`

```markdown
---
description: Cash flow statement and runway analysis
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period]
---

Generate a Cash Flow statement. If $ARGUMENTS is provided, use it as
the period. Default to current month.

Steps:
1. Use qbReports with reportType "CashFlow" for the requested period
2. Break down by section:
   - Operating Activities (net income + adjustments)
   - Investing Activities (asset purchases/sales)
   - Financing Activities (loans, owner draws/contributions)
3. Calculate monthly burn rate from operating cash flow
4. Get current cash balance from qbReports BalanceSheet
5. Estimate cash runway: current cash balance / average monthly burn
6. Compare to prior period — is cash position improving or declining?
7. Flag if runway is below 6 months
8. Present: cash flow summary, burn rate, runway estimate, trend
```

---

### 4.8 `commands/record.md`

```markdown
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
   - Adjusting entry → qbJournalEntry
7. Confirm success with transaction ID and summary

Special rules:
- For purchases over $5,000: ask "Should this be recorded as a fixed asset or expensed?"
- For payments: check for outstanding invoices/bills to link against first
- Suggest class, memo, and reference number for audit trail but don't require them
```

---

### 4.9 `commands/aging.md`

```markdown
---
description: AR/AP aging report — who owes you, what you owe
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [receivables|payables|both]
---

Generate aging report. Default to both AR and AP if $ARGUMENTS is empty.

Steps:
1. Determine type from $ARGUMENTS:
   - "receivables", "ar", "who owes" → AgedReceivables only
   - "payables", "ap", "what we owe" → AgedPayables only
   - anything else or empty → both
2. Use qbReports with "AgedReceivables" and/or "AgedPayables"
3. Summarize by aging bucket:
   - Current (not yet due)
   - 1-30 days past due
   - 31-60 days past due
   - 61-90 days past due
   - 90+ days past due
4. List top overdue items with customer/vendor name and amount
5. Calculate:
   - Total outstanding
   - Average days outstanding
   - Percentage in each bucket
6. Flag items >60 days as needing follow-up
7. For large overdue amounts, note the specific invoices/bills
8. Present: aging summary table, overdue highlights, action items
```

---

### 4.10 `commands/find.md`

```markdown
---
description: Search transactions by vendor, customer, date, or amount
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: <search criteria>
---

Search for transactions matching: "$ARGUMENTS"

Steps:
1. Parse search criteria from the user's input:
   - Vendor or customer name
   - Date or date range
   - Amount or amount range
   - Transaction type (expense, bill, invoice, etc.)
   - Account or category
2. Use qbFetchTransactions with appropriate filters
3. If too many results, narrow by asking the user for more specific criteria
4. Display results in a clear table format:
   - Date
   - Type (Bill, Expense, Invoice, etc.)
   - Vendor/Customer
   - Amount
   - Account
   - Transaction ID
5. If user wants to act on a result (void, pay, edit), guide them to the
   appropriate next step
```

---

### 4.11 `skills/bookkeeping/SKILL.md`

```markdown
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
```

---

### 4.12 `skills/bookkeeping/references/transaction-workflows.md`

```markdown
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
```

---

### 4.13 `skills/bookkeeping/references/duplicate-detection.md`

```markdown
# Duplicate Detection — Methodology and Procedures

## Why Check for Duplicates

Double-recording transactions is one of the most common bookkeeping errors.
It inflates expenses, revenue, or both, leading to incorrect financial
statements. Always check before creating.

## When to Check

Before creating ANY of these transaction types:
- qbBill
- qbExpense
- qbInvoice
- qbSalesReceipt
- qbJournalEntry
- qbDeposit
- qbRefundReceipt

Do NOT need to check for:
- qbBillPayment (linked to specific bill)
- qbReceivePayment (linked to specific invoice)
- qbVoidTransaction (modifying existing)

## How to Check

### Step 1: Query Existing Transactions

Use qbFetchTransactions with these filters:
- **Vendor/Customer**: Same entity name
- **Date range**: ±3 days from the intended transaction date
- **Transaction type**: Same type if possible

### Step 2: Compare Results

A potential duplicate exists if ALL of these match:
- Same vendor or customer (exact or fuzzy name match)
- Same date (±1 day for potential timezone differences)
- Same total amount (±5% for rounding/tax differences)

### Step 3: Present to User

If potential duplicates found, show them in a table:
```
Existing transactions that may be duplicates:
| Date       | Type    | Vendor      | Amount  | ID    |
|------------|---------|-------------|---------|-------|
| 2026-03-13 | Expense | Staples     | $150.00 | #1234 |
```

Then ask: "A similar transaction already exists. Is this a new,
separate transaction?"

### Step 4: Proceed or Abort

- If user confirms it's new → proceed with creation
- If user says it's a duplicate → abort and reference the existing one

## Fuzzy Name Matching

Vendor/customer names may not match exactly:
- "Office Depot" vs "Office Depot Inc"
- "Amazon" vs "Amazon.com" vs "AMZN"
- "John's Plumbing" vs "Johns Plumbing LLC"

When searching, use partial name matches. If the vendor/customer lookup
returns multiple similar results, list them all.

## Amount Tolerance

Allow ±5% tolerance because:
- Tax may or may not be included
- Shipping charges may vary
- Currency rounding differences
- Partial payments

## Common Duplicate Scenarios

1. **Bank feed + manual entry**: User records an expense, then bank feed
   imports the same transaction
2. **Invoice + payment confusion**: User creates an invoice, then also
   records it as a sale
3. **Bill + expense confusion**: User records a bill, then also records
   the payment as a separate expense
4. **Recurring transactions**: Monthly rent/subscription recorded twice
```

---

### 4.14 `skills/bookkeeping/references/categorization.md`

```markdown
# Account Categorization — Rules and Patterns

## Categorization Consistency

The most important rule: categorize consistently. If "Staples" purchases
have been going to "Office Supplies" (AcctNum 6000), new Staples purchases
should also go to "Office Supplies" unless the user explicitly says otherwise.

## How to Maintain Consistency

### Step 1: Check Recent History

Before proposing an account for a transaction, fetch the vendor/customer's
recent transactions via qbFetchTransactions. Look at which accounts were
used in the last 3-5 transactions.

### Step 2: Follow the Pattern

If the last 3 transactions for "Staples" all used "Office Supplies":
- Default to "Office Supplies" in the proposal
- Show the user: "Based on previous transactions, categorizing to
  Office Supplies (6000). Change?"

### Step 3: Ask When Uncertain

If there's no history, or if history is inconsistent, ask the user.
Never guess. Show them the available account options from qbMasterData.

## Common Account Mappings

These are typical mappings for small businesses. Always verify against
the specific company's chart of accounts via qbMasterData.

### Expense Accounts (typical)
| Category | Common Vendors | Typical AcctNum Range |
|----------|----------------|----------------------|
| Office Supplies | Staples, Office Depot, Amazon | 6000-6099 |
| Rent/Lease | Landlord, Property Management | 6100-6199 |
| Utilities | Electric co, Gas co, Water, Internet | 6200-6299 |
| Insurance | Insurance carriers | 6300-6399 |
| Professional Services | Lawyers, CPAs, Consultants | 6400-6499 |
| Software/SaaS | Tech vendors | 6500-6599 |
| Travel | Airlines, Hotels, Uber | 6600-6699 |
| Meals & Entertainment | Restaurants | 6700-6799 |
| Advertising/Marketing | Google, Facebook, agencies | 6800-6899 |
| Payroll | Payroll providers | 6900-6999 |

### Income Accounts (typical)
| Category | Typical AcctNum Range |
|----------|----------------------|
| Service Revenue | 4000-4099 |
| Product Sales | 4100-4199 |
| Consulting Income | 4200-4299 |
| Other Income | 4900-4999 |

### Balance Sheet Accounts (typical)
| Category | Typical AcctNum Range |
|----------|----------------------|
| Checking/Savings | 1000-1099 |
| Accounts Receivable | 1100-1199 |
| Inventory | 1200-1299 |
| Fixed Assets | 1500-1599 |
| Accounts Payable | 2000-2099 |
| Credit Cards | 2100-2199 |
| Loans | 2500-2599 |
| Equity | 3000-3099 |

## When Account Doesn't Exist

If the appropriate account doesn't exist in the chart of accounts:
1. Suggest the closest existing account
2. Ask if user wants to create a new account via qbMasterData (operation: 'create')
3. If creating, suggest an appropriate AcctNum based on the category ranges above

## Account Types

QuickBooks account types determine where they appear on financial statements:
- **Income** → P&L (revenue section)
- **Expense** → P&L (expense section)
- **Cost of Goods Sold** → P&L (COGS section)
- **Bank** → Balance Sheet (assets)
- **Accounts Receivable** → Balance Sheet (assets)
- **Other Current Asset** → Balance Sheet (assets)
- **Fixed Asset** → Balance Sheet (assets)
- **Accounts Payable** → Balance Sheet (liabilities)
- **Credit Card** → Balance Sheet (liabilities)
- **Other Current Liability** → Balance Sheet (liabilities)
- **Long Term Liability** → Balance Sheet (liabilities)
- **Equity** → Balance Sheet (equity)
```

---

### 4.15 `skills/financial-analysis/SKILL.md`

```markdown
---
name: Financial Analysis
description: >
  This skill should be used when the user asks to "show me the P&L",
  "how is my business doing", "what are my expenses", "revenue trends",
  "cash flow analysis", "compare this month to last month", "aging report",
  "who owes me money", "what bills are due", "financial health check",
  "variance analysis", "margin analysis", "cash runway", or any financial
  reporting and analysis task.
version: 1.0.0
---

## Purpose

Provide financial analysis expertise for interpreting QuickBooks reports
and surfacing actionable business insights.

## Analysis Framework

### Always Compare Periods

Never present numbers in isolation. Always compare:
- Current month vs prior month
- Current quarter vs prior quarter
- Year-to-date vs prior year-to-date

Highlight variances with both % change and $ impact.

### Report Types

| Report | Tool Call | Use When |
|--------|-----------|----------|
| Profit & Loss | qbReports (ProfitAndLoss) | Revenue, expenses, net income |
| Balance Sheet | qbReports (BalanceSheet) | Assets, liabilities, equity |
| Cash Flow | qbReports (CashFlow) | Cash position and movement |
| AR Aging | qbReports (AgedReceivables) | Who owes money |
| AP Aging | qbReports (AgedPayables) | Bills due |
| Trial Balance | qbReports (TrialBalance) | Account balances |
| Transaction List | qbReports (TransactionList) | Detailed transactions |
| Customer Income | qbReports (CustomerIncome) | Revenue by customer |
| Vendor Expenses | qbReports (VendorExpenses) | Spend by vendor |

### Translate to Business Language

Convert accounting numbers into plain business insights:
- Not: "Revenue decreased 12% QoQ"
- Instead: "Revenue dropped $15K (12%) from last quarter — driven mainly
  by a $12K decline in consulting income. Worth investigating whether
  this is seasonal or if a key client reduced their engagement."

### Proactive Warning Signs

Surface these automatically when detected:
- **Cash runway** — If current burn rate exceeds cash reserves
- **Margin compression** — Gross margin dropping >3% period-over-period
- **Aging receivables** — Invoices >60 days outstanding
- **Concentration risk** — Single customer >30% of revenue
- **Expense spikes** — Any category up >25% without obvious explanation

### Back Up Insights with Data

For any major fluctuating account, fetch transaction-level detail using
qbFetchTransactions or qbReports (TransactionList) to identify the
specific transactions driving the variance.

## Additional Resources

### Reference Files

- **`references/variance-analysis.md`** — Detailed methodology for
  period-over-period comparison, materiality thresholds, and narrative
  templates
- **`references/warning-signs.md`** — Comprehensive list of financial
  red flags with detection logic and recommended actions
```

---

### 4.16 `skills/financial-analysis/references/variance-analysis.md`

```markdown
# Variance Analysis — Methodology and Templates

## Period Comparison Framework

### Which Periods to Compare

| User Request | Primary Comparison | Secondary Comparison |
|-------------|-------------------|---------------------|
| "This month" | Current month vs prior month | vs same month last year |
| "This quarter" | Current quarter vs prior quarter | vs same quarter last year |
| "This year" | YTD vs prior YTD | Full prior year |
| "How am I doing" | Current month vs prior month | + YTD vs prior YTD |

### Materiality Thresholds

Not every variance deserves commentary. Focus on variances that are:

**Material by dollar amount:**
- Revenue line items: >$5,000 change
- Expense categories: >$2,000 change
- Net income: any significant change

**Material by percentage:**
- >10% change on large accounts (>$10K)
- >25% change on medium accounts ($1K-$10K)
- >50% change on small accounts (<$1K)

**Always material regardless of size:**
- New accounts that didn't exist in prior period
- Accounts that dropped to zero
- Sign changes (positive to negative or vice versa)

### Variance Calculation

```
$ Change = Current Period - Prior Period
% Change = (Current - Prior) / |Prior| × 100

If Prior = 0 and Current > 0: "New this period"
If Prior > 0 and Current = 0: "None this period (was $X)"
```

## Narrative Templates

### Revenue Increase
"Revenue grew [$ amount] ([%]) to [$total] this [period]. The increase was
primarily driven by [top contributor] ([$ amount]). [Additional context about
customer mix, new clients, or seasonal factors if visible from data]."

### Revenue Decrease
"Revenue declined [$ amount] ([%]) to [$total] this [period]. The main driver
was [top contributor] ([$ amount]). [Recommendation: investigate whether this
is seasonal, client-specific, or a broader trend. Consider pulling CustomerIncome
report for detailed breakdown.]"

### Expense Increase
"[Category] expenses increased [$ amount] ([%]) to [$total]. [If specific large
transaction visible: 'This was primarily due to [description].' If not:
'Recommend reviewing individual transactions to understand the increase.']"

### Margin Compression
"Gross margin declined from [prior]% to [current]% — a [change]pp compression.
This means [$ amount] less profit per dollar of revenue. [Recommend investigating
COGS increases or pricing changes.]"

### Net Income Impact
"Net income [increased/decreased] by [$ amount] ([%]). Key drivers:
1. [Largest contributing factor] ([±$ amount])
2. [Second factor] ([±$ amount])
[Overall: business is [improving/stable/declining] relative to prior period.]"

## Drill-Down Process

When a variance is material:

1. Identify the account(s) driving the variance
2. Use qbReports (TransactionList) filtered by that account and date range
3. Look for:
   - Large individual transactions (>50% of the variance)
   - New recurring charges
   - Missing recurring revenue
   - One-time items that distort the trend
4. Include specific transaction references in the narrative

## Multi-Period Trends

When the user asks "how am I doing" or wants a health check:

1. Pull 3-6 months of P&L data
2. Calculate month-over-month trends for:
   - Total revenue
   - Gross margin %
   - Total operating expenses
   - Net income
3. Identify direction: growing, stable, or declining
4. Calculate growth rates: monthly and annualized
5. Present as a trend narrative, not just numbers
```

---

### 4.17 `skills/financial-analysis/references/warning-signs.md`

```markdown
# Financial Warning Signs — Detection and Response

## Cash Position Warnings

### Low Cash Runway

**Detection:** Monthly burn rate (negative operating cash flow) exceeds
cash reserves when projected forward.

**Calculation:**
```
Monthly burn = Average of last 3 months' operating cash flow (if negative)
Current cash = Total bank account balances from Balance Sheet
Runway = Current cash / |Monthly burn|
```

**Thresholds:**
- <3 months: CRITICAL — immediate action needed
- 3-6 months: WARNING — plan for funding or cost reduction
- 6-12 months: MONITOR — healthy but watch trend
- >12 months: OK

**Response:** "Current cash runway is approximately [X] months based on
average monthly burn of [$Y]. [Recommendation based on threshold.]"

### Declining Cash Balance

**Detection:** Cash balance decreased >15% month-over-month for 2+
consecutive months.

**Response:** "Cash balance has declined for [N] consecutive months,
from [$X] to [$Y]. This trend suggests [spending exceeds revenue /
large one-time expenses / AR collection issues]. Recommend reviewing
cash flow statement for details."

## Profitability Warnings

### Gross Margin Compression

**Detection:** Gross margin % dropped >3 percentage points from prior period.

**Calculation:**
```
Gross Margin % = (Revenue - COGS) / Revenue × 100
Change = Current GM% - Prior GM%
```

**Response:** "Gross margin declined from [X]% to [Y]%, a [Z]pp compression.
This means [$amount] less profit per dollar of revenue. Investigate: are
input costs rising? Has pricing changed? Is the product/service mix shifting?"

### Negative Net Income

**Detection:** Net income is negative (net loss).

**Response:** "The business reported a net loss of [$X] this period.
[If first time: 'This is a change from profitable last period — investigate
what changed.' If ongoing: 'This is the [Nth] consecutive month of losses.
Current accumulated loss: [$Y].']"

### Operating Expense Growth Exceeding Revenue

**Detection:** OpEx growth rate > Revenue growth rate for 2+ periods.

**Response:** "Operating expenses are growing faster than revenue —
expenses up [X]% vs revenue up [Y]%. If this trend continues, margins
will continue to compress. Review expense categories for optimization."

## Receivables Warnings

### Aging AR

**Detection:** Receivables >60 days represent >20% of total AR.

**Response:** "[$X] ([Y]%) of accounts receivable is over 60 days past due.
Top overdue accounts: [list]. Recommend: follow up on collections, consider
offering early payment discounts, review credit terms."

### Growing AR Relative to Revenue

**Detection:** AR / Monthly Revenue ratio increasing over 3+ months.

**Calculation:**
```
Days Sales Outstanding (DSO) = (AR / Revenue) × Days in Period
```

**Response:** "Days Sales Outstanding has increased from [X] to [Y] days
over the past [N] months, meaning customers are taking longer to pay.
This ties up working capital. Current AR: [$X]."

### Single Customer Concentration

**Detection:** Any single customer >30% of total revenue (from CustomerIncome report).

**Response:** "[$Customer] represents [X]% of total revenue ([$Y] of [$Z]).
This creates concentration risk — if this customer reduces spending or
leaves, it would significantly impact the business. Consider diversifying
the customer base."

## Payables Warnings

### Overdue Payables

**Detection:** AP >60 days represents >30% of total AP.

**Response:** "[$X] in payables is over 60 days past due. This may damage
vendor relationships, result in late fees, or indicate cash flow problems.
Top overdue: [list]. Prioritize payment or negotiate extended terms."

### Rising AP

**Detection:** AP increased >25% month-over-month without corresponding
revenue increase.

**Response:** "Accounts payable increased [X]% this month to [$Y] without
a corresponding revenue increase. This suggests the business is stretching
payments — monitor cash position."

## Expense Warnings

### Category Spike

**Detection:** Any expense category increased >25% from prior period
AND the increase is >$2,000.

**Response:** "[Category] expenses jumped [X]% ([$Y]) this period.
[If specific large transaction found: 'Driven by: [description].'
If not: 'Review transactions in this category for unusual items.']"

### New Large Expense

**Detection:** A new expense account appeared that wasn't used in the
prior period, with >$5,000 in activity.

**Response:** "New expense category '[Account]' appeared this period
with [$X] in charges. This wasn't present last period. Verify this is
correctly categorized and expected."

## When to Surface Warnings

- **Always** during financial health checks ("how is my business doing")
- **Always** in P&L analysis when the warning condition is detected
- **Proactively** mention cash runway in any cash-related analysis
- **Proactively** mention aging in any AR/AP report
- Do NOT repeatedly warn about the same issue within a single conversation
```

---

### 4.18 `agents/accountant.md`

```markdown
---
name: accountant
description: >
  Use this agent when the user needs to record, modify, or void QuickBooks
  transactions. Handles expense recording, bill creation, invoice generation,
  payment recording, journal entries, and transaction management.
  <example>
  Context: User wants to record a business expense
  user: "I paid $250 to Office Depot for supplies on March 10"
  assistant: "Delegating to the accountant agent to record this expense."
  <commentary>
  Transaction recording with vendor lookup, duplicate check, and confirmation.
  </commentary>
  </example>
  <example>
  Context: User wants to pay an outstanding bill
  user: "Pay the bill from Acme Corp"
  assistant: "Delegating to the accountant agent to process this payment."
  <commentary>
  Requires looking up outstanding bills, confirming which one, and recording payment.
  </commentary>
  </example>
model: inherit
color: green
tools: ["mcp__plugin_deepledger_deepledger__*"]
---

You are an expert bookkeeper. Your role is to record QuickBooks transactions
accurately and completely.

**Operating Framework:** Analyze → Propose → Confirm → Execute

**Your Core Responsibilities:**
1. Look up master data before proposing any transaction
2. Check for duplicate transactions before creating new ones
3. Always display account codes (AcctNum) and IDs when proposing
4. Wait for user confirmation before any write operation
5. Link payments to existing invoices/bills when possible
6. Suggest class, reference, and memo fields for audit trails

**Process for Every Transaction:**
1. Identify transaction type from user's description
2. Fetch relevant vendors/customers/accounts via qbMasterData
3. Check duplicates via qbFetchTransactions
4. Propose with full details (account name, AcctNum, Account ID, amounts)
5. Get explicit user confirmation
6. Execute and confirm success with transaction ID

**Tool Selection:**
- Paid expense (card/check/cash) → qbExpense
- Vendor bill (unpaid) → qbBill
- Pay a bill → qbBillPayment
- Customer invoice → qbInvoice
- Receive payment → qbReceivePayment
- Cash sale → qbSalesReceipt
- Bank deposit → qbDeposit
- Refund → qbRefundReceipt
- Adjusting entry → qbJournalEntry
- Cancel transaction → qbVoidTransaction
- Attach receipt → qbGetUploadUrl

**Capitalization Rule:**
Purchases over $5,000 — ask if it should be a fixed asset or expense.

**Payment Rule:**
Before recording any payment, check for outstanding invoices/bills that
should be linked. Do not create standalone payment transactions when there
are open documents to apply them against.
```

---

### 4.19 `agents/cfo.md`

```markdown
---
name: cfo
description: >
  Use this agent when the user needs financial analysis, reporting, or
  strategic business insights from their QuickBooks data. Handles P&L
  analysis, cash flow review, aging reports, trend analysis, and
  financial health assessments.
  <example>
  Context: User wants to understand their financial position
  user: "How is my business doing this quarter?"
  assistant: "Delegating to the CFO agent for a comprehensive financial review."
  <commentary>
  Requires pulling multiple reports, comparing periods, and synthesizing insights.
  </commentary>
  </example>
  <example>
  Context: User wants to understand expense trends
  user: "Why are my expenses up this month?"
  assistant: "Delegating to the CFO agent to analyze expense variances."
  <commentary>
  Needs P&L comparison, transaction-level drill-down, and narrative explanation.
  </commentary>
  </example>
model: inherit
color: cyan
tools: ["mcp__plugin_deepledger_deepledger__*"]
---

You are an experienced CFO providing financial analysis and strategic insights.

**Your Core Responsibilities:**
1. Always compare periods — never present numbers in isolation
2. Translate numbers into plain business language
3. Proactively surface warning signs before they become problems
4. Back up every insight with transaction-level data
5. Provide actionable recommendations, not just observations

**Analysis Process:**
1. Pull relevant reports via qbReports
2. Pull comparison period data
3. Calculate variances (% and $ impact)
4. For major variances (>$5K or >20%), drill into transactions via
   qbFetchTransactions or qbReports (TransactionList)
5. Synthesize findings into narrative with recommendations

**Available Reports:**
- ProfitAndLoss — revenue, expenses, net income
- BalanceSheet — assets, liabilities, equity
- CashFlow — cash movement by activity
- AgedReceivables — who owes money, by aging bucket
- AgedPayables — what bills are due
- TrialBalance — all account balances
- TransactionList — individual transactions with filters
- CustomerIncome — revenue breakdown by customer
- VendorExpenses — spend breakdown by vendor

**Warning Signs to Surface Automatically:**
- Cash runway below 6 months
- Gross margin compression >3% period-over-period
- Receivables aging >60 days
- Single customer >30% of revenue
- Any expense category up >25% without explanation
- Net losses for 2+ consecutive months
- Operating expenses growing faster than revenue

**Output Style:**
Lead with the headline insight, then supporting data, then recommendation.
Use tables for numbers. Keep language accessible to non-accountants.
Do not use jargon without explaining it.
```

---

### 4.20 `hooks/hooks.json`

```json
{
  "description": "DeepLedger safety hooks — validate write operations and prevent duplicates",
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__plugin_deepledger_deepledger__qbBill|mcp__plugin_deepledger_deepledger__qbExpense|mcp__plugin_deepledger_deepledger__qbInvoice|mcp__plugin_deepledger_deepledger__qbSalesReceipt|mcp__plugin_deepledger_deepledger__qbJournalEntry|mcp__plugin_deepledger_deepledger__qbDeposit|mcp__plugin_deepledger_deepledger__qbRefundReceipt",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "A write operation is about to execute. Verify: (1) Was qbMasterData called first to verify accounts? (2) Was qbFetchTransactions called to check for duplicates? (3) Did the user explicitly confirm this transaction? If any check fails, return 'block' with the reason. If all pass, return 'approve'.",
            "timeout": 15
          }
        ]
      },
      {
        "matcher": "mcp__plugin_deepledger_deepledger__qbVoidTransaction",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "A void/delete operation is about to execute. This is destructive and irreversible. Verify the user explicitly confirmed voiding this specific transaction by ID. If not confirmed, return 'block'. If confirmed, return 'approve'.",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

---

## 5. Implementation Checklist

### Phase 1: Minimal Plugin (get it working)

```bash
mkdir -p deepledger-plugin/.claude-plugin
# Create plugin.json, .mcp.json, .gitignore
# Test: claude --plugin-dir ./deepledger-plugin
# Verify: OAuth flow works, tools appear in /mcp output
```

- [ ] Create `.claude-plugin/plugin.json`
- [ ] Create `.mcp.json`
- [ ] Create `.gitignore`
- [ ] Test OAuth flow and MCP connection
- [ ] Verify tools appear

### Phase 2: Skills

- [ ] Create `skills/bookkeeping/SKILL.md`
- [ ] Create `skills/bookkeeping/references/transaction-workflows.md`
- [ ] Create `skills/bookkeeping/references/duplicate-detection.md`
- [ ] Create `skills/bookkeeping/references/categorization.md`
- [ ] Create `skills/financial-analysis/SKILL.md`
- [ ] Create `skills/financial-analysis/references/variance-analysis.md`
- [ ] Create `skills/financial-analysis/references/warning-signs.md`
- [ ] Test: ask bookkeeping questions → verify skills trigger

### Phase 3: Commands

- [ ] Create `commands/pnl.md`
- [ ] Create `commands/balance-sheet.md`
- [ ] Create `commands/cash-flow.md`
- [ ] Create `commands/record.md`
- [ ] Create `commands/aging.md`
- [ ] Create `commands/find.md`
- [ ] Test: run each `/command` and verify behavior

### Phase 4: Agents

- [ ] Create `agents/accountant.md`
- [ ] Create `agents/cfo.md`
- [ ] Test: verify agents trigger on relevant requests

### Phase 5: Hooks

- [ ] Create `hooks/hooks.json`
- [ ] Test: write without confirmation → verify hook blocks
- [ ] Test: void without confirmation → verify hook blocks

### Phase 6: Polish and Distribute

- [ ] Create `README.md`
- [ ] Init git repo, push to GitHub
- [ ] Test full install flow from clean clone

---

## 6. Testing the Plugin

```bash
# Start with the plugin
claude --plugin-dir /path/to/deepledger-plugin

# Verify MCP connection
> /mcp
# Should show "deepledger" server with all qb* tools

# Test skill trigger
> "Record an expense for $100 to Amazon"
# Should: look up vendor, check duplicates, propose, confirm, create

# Test command
> /pnl
# Should: generate P&L with variance analysis

# Test agent
> "How is my business doing?"
# Should: delegate to CFO agent, pull reports, synthesize insights

# Test hooks
> "Create an invoice for $5000" (then try to skip confirmation)
# Should: hook blocks if master data wasn't checked
```
