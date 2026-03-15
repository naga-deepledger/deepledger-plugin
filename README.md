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
