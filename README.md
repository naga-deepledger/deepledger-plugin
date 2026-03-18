# DeepLedger Plugin for Claude Code

Autonomous AI bookkeeper with QuickBooks Online integration. Record
transactions, generate financial reports, and get actionable business
insights — all through natural language with minimal back-and-forth.

DeepLedger acts like an expert bookkeeper: it infers transaction types,
auto-categorizes expenses based on vendor history, checks for duplicates
silently, and only asks questions when the answer would affect accounting
accuracy.

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
| `/record` | Record a transaction (expense, bill, invoice, payment, etc.) |
| `/transfer` | Transfer money between bank accounts or pay a credit card |
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

## How It Works

### Autonomous Bookkeeping

Just describe what happened — DeepLedger handles the rest:

```
"I paid $250 to Staples for office supplies"
```

DeepLedger will automatically:
- Identify this as a paid expense (→ qbExpense)
- Find "Staples" in your vendor list
- Check Staples' transaction history to determine the right expense account
- Default to today's date
- Check for duplicate transactions (silently — no noise if none found)
- Present a ready-to-confirm proposal with all details filled in

You just confirm and it's recorded.

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

All write operations follow a strict **Analyze → Propose → Confirm → Execute**
workflow:

- Master data is verified before proposing any transaction
- Duplicate checks run automatically (silently if no matches found)
- You must explicitly confirm before any transaction is recorded
- Void/delete operations require additional explicit confirmation
- Purchases over $5,000 trigger an asset-vs-expense question

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
