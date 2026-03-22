# DeepLedger Plugin for Claude Code

Autonomous AI bookkeeper with QuickBooks Online integration. Record
transactions, generate financial reports, and get actionable business
insights — all through natural language with minimal back-and-forth.

DeepLedger acts like an expert autonomous bookkeeper: it infers transaction
types, auto-categorizes expenses based on vendor history, checks for
duplicates, verifies source documents, and records transactions directly.
It only escalates when information is missing or accuracy is at risk.
Every transaction is document-backed and audit-ready.

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

### API-Powered Commands (fast, reliable)

These use the QuickBooks API via DeepLedger — no browser needed.

| Command | Description |
|---------|-------------|
| `/record` | Record a transaction (expense, bill, invoice, payment, etc.) |
| `/transfer` | Transfer money between bank accounts or pay a credit card |
| `/estimate` | Create an estimate or quote for a customer |
| `/purchase-order` | Create a purchase order for a vendor |
| `/pnl` | Profit & Loss with variance analysis |
| `/balance-sheet` | Balance sheet snapshot with key ratios |
| `/cash-flow` | Cash flow statement and runway analysis |
| `/aging` | AR/AP aging report |
| `/find` | Search transactions |
| `/health-check` | Comprehensive financial health assessment |
| `/close-books` | Month-end close checklist and validation |
| `/budget` | Budget vs actual analysis (uses historical baseline) |
| `/vendor-expenses` | Vendor spend analysis and trends |
| `/customer-income` | Customer revenue analysis and concentration |

### Browser-Powered Commands (requires QBO login)

These features have NO QuickBooks API — they use browser automation to
interact with the QBO UI directly. Requires the user to be logged into
QuickBooks Online.

| Command | Description |
|---------|-------------|
| `/reconcile` | Reconcile a bank or credit card account |
| `/recurring` | Create, list, edit, or delete recurring transactions |
| `/bank-rules` | Manage bank feed auto-categorization rules |

## Architecture

DeepLedger uses two integration channels:

```
User Request
    ↓
Plugin decides: API or UI task?
    ↓
┌─────────────────┐    ┌──────────────────┐
│ DeepLedger MCP  │    │ Chrome MCP       │
│ (QBO API)       │    │ (Browser)        │
├─────────────────┤    ├──────────────────┤
│ Transactions    │    │ Reconciliation   │
│ Reports         │    │ Bank Feeds       │
│ Master Data     │    │ Recurring Txns   │
│ Attachments     │    │ Bank Rules       │
│ Estimates & POs │    │ Budgets (native) │
└────────┬────────┘    └────────┬─────────┘
         │                      │
         ▼                      ▼
     QBO REST API          QBO Web UI
```

**API-powered** (fast, reliable): All transaction recording, reporting,
and master data management. This covers ~90% of daily bookkeeping.

**Browser-powered** (slower, requires login): Bank reconciliation, recurring
transactions, bank rules, and other features that Intuit has not exposed
via API. These require the user to be logged into QuickBooks Online.

## How It Works

### Autonomous Bookkeeping

Just describe what happened — DeepLedger handles the rest:

```
"I paid $250 to Staples for office supplies"
```

DeepLedger will automatically:
- Identify this as a paid expense (→ qbExpense)
- Verify source document exists (receipt, bank feed, or user description)
- Find "Staples" in your vendor list
- Check Staples' transaction history to determine the right expense account
- Default to today's date
- Check for duplicate transactions
- Record the transaction directly (no confirmation needed)
- Attach the source document to the QB transaction
- Log the action for audit trail

### Smart Inference

DeepLedger infers intelligently and only asks when it matters:

| Situation | What Happens |
|-----------|-------------|
| Single vendor match | Auto-selected, no question |
| Vendor has history | Same account used, no question |
| Date not specified | Defaults to today, no question |
| No duplicates found | Proceeds silently, no question |
| Multiple vendor matches | Asks which one (accounting accuracy) |
| Purchase over $5,000 | Asks: asset or expense? (accounting accuracy) |
| Potential duplicate found | Shows match, asks if new (prevents errors) |

### Autonomous Financial Analysis

Ask any financial question and get a complete answer in one shot:

```
"How is my business doing this quarter?"
```

DeepLedger will pull P&L, Balance Sheet, Cash Flow, and Aging reports,
compare periods, calculate key metrics, drill into major variances,
check for warning signs, and present everything with actionable
recommendations — no follow-up prompts needed.

## Examples

**Recording transactions:**
```
"I paid $250 to Staples for office supplies"
"Got a $3,000 bill from our landlord for March rent"
"Invoice Acme Corp $5,000 for consulting services"
"Pay the outstanding bill from Office Depot"
"Transfer $5,000 from savings to checking"
"Pay off the credit card from our checking account"
"Issue a credit memo to Johnson Inc for $300"
"Record 5 transactions: $50 Uber, $120 dinner at Nobu, $200 AWS, $85 Zoom, $45 Staples"
```

**Financial analysis:**
```
"How is my business doing?"
"Show me the P&L"
"What's our cash runway?"
"Who owes us money?"
"Why are expenses up this month?"
"Which vendors are we spending the most with?"
"Show me revenue by customer for Q1"
"Are we on budget this month?"
"Close the books for February"
```

**Searching:**
```
"Find all payments to Amazon this quarter"
"Show me transactions over $1,000 this month"
"What did we spend on software?"
```

## Safety Features

All write operations follow a strict **Analyze → Verify → Execute → Log**
workflow:

- Every transaction must be backed by a source document (receipt, bank feed,
  invoice, email, or detailed user description)
- Master data is verified before any transaction is recorded
- Duplicate checks run automatically before every write
- Confidence scoring determines action: ≥80% = execute, 60-79% = execute + flag
  for review, <60% = escalate to human via contactHuman
- Void/delete operations require human confirmation via contactHuman
- Purchases over $5,000 trigger escalation for asset-vs-expense decision
- Source documents are attached to QB transactions after recording
- Every action is logged via agentLog for complete audit trail

## Requirements

- [Claude Code](https://claude.ai/code) installed
- A DeepLedger account with QuickBooks Online connected

## Troubleshooting

**Authentication issues:** If the browser window doesn't open or
authentication fails, restart Claude Code and try again. Ensure your
DeepLedger account has an active QuickBooks Online connection.

**"Vendor/customer not found":** The plugin searches by partial name match.
Try a shorter or different variation of the name. You can also ask to
create a new vendor or customer.

**Duplicate warnings:** The plugin checks for existing transactions with
similar dates, amounts, and entities. If you're sure a transaction is new,
confirm when prompted and it will proceed.

**Need help?** Contact support@deepledger.ai
