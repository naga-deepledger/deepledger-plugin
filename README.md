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
| `/record` | Record a transaction (expense, bill, invoice, payment, etc.) |
| `/pnl` | Profit & Loss with variance analysis |
| `/balance-sheet` | Balance sheet snapshot with key ratios |
| `/cash-flow` | Cash flow statement and runway analysis |
| `/aging` | AR/AP aging report |
| `/find` | Search transactions |
| `/health-check` | Comprehensive financial health assessment |
| `/vendor-expenses` | Vendor spend analysis and trends |
| `/customer-income` | Customer revenue analysis and concentration |

## Examples

**Recording transactions:**
```
"Record a $500 expense to Office Depot for supplies on March 10"
"Create an invoice for Acme Corp — $2,500 for consulting services"
"Pay the outstanding bill from our landlord"
"Issue a credit memo to Johnson Inc for $300"
```

**Financial analysis:**
```
"Show me this month's P&L compared to last month"
"How is my business doing this quarter?"
"Who owes us money?"
"What's our cash runway?"
"Which vendors are we spending the most with?"
"Show me revenue by customer for Q1"
```

**Searching and investigating:**
```
"Find all transactions over $1,000 this month"
"Show me all payments to Amazon in the last 90 days"
"What did we spend on software this quarter?"
```

## Safety Features

All write operations (recording expenses, creating invoices, etc.) follow a
strict **Analyze → Propose → Confirm → Execute** workflow:

- Master data is verified before proposing any transaction
- Duplicate checks run automatically before creating entries
- You must explicitly confirm before any transaction is recorded
- Void/delete operations require additional explicit confirmation

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
