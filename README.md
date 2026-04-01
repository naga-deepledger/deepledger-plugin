# DeepLedger Plugin

Claude Code plugin for autonomous AI bookkeeping and financial analysis powered by QuickBooks Online.

## Prerequisites

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- A DeepLedger account with QuickBooks connected via the [portal](https://deepledger.ai)
- MCP server running at `https://deepledger-mcp.onrender.com/mcp`

## Setup

```bash
# Run Claude Code with the plugin
claude --plugin-dir ./deepledger-plugin
```

The plugin connects to the DeepLedger MCP server using HTTP Streamable transport. Authentication is handled via Supabase Bearer tokens — you'll be prompted to authenticate on first use.

## Quick Start

```bash
# 1. Bootstrap — learn from existing QB history (run once per client)
/bootstrap

# 2. Process bank feed — agent already knows your vendors
/bank-feed

# 3. Record a transaction
/record Paid $500 to Office Depot for office supplies with company credit card

# 4. Generate a P&L
/pnl

# 5. Run the autonomous bookkeeping loop
/loop
```

> **First time?** Always run `/bootstrap` first. It reads your existing QuickBooks transactions and learns vendor/account mappings so the agent is accurate from day one. Without it, every vendor is unknown and gets flagged for CPA review.

## Commands

| Command | Description |
|---------|-------------|
| `/bootstrap` | Learn from existing QB history (run once per client) |
| `/record <description>` | Record a financial transaction |
| `/bank-feed` | Process unrecorded bank/CC transactions |
| `/find <query>` | Search transactions, vendors, customers, accounts |
| `/transfer <amount> from A to B` | Transfer between own bank accounts |
| `/pnl [period]` | Profit & Loss report |
| `/balance-sheet [date]` | Balance Sheet report |
| `/cash-flow [period]` | Cash Flow Statement |
| `/aging [ar\|ap]` | Aged Receivables / Payables |
| `/close-books [month]` | Month-end close workflow |
| `/health-check [--detailed]` | Financial health scorecard |
| `/loop` | Autonomous bookkeeping cycle |
| `/budget [period]` | Budget vs actuals comparison |
| `/recurring [list\|create\|pause]` | Manage recurring transactions |

## Agents

- **Accountant** — Day-to-day bookkeeping: transaction recording, bank feed processing, reconciliation, month-end close
- **CFO** — Strategic analysis: financial reports, ratio analysis, trend identification, health monitoring

## Safety Model

Every QuickBooks write operation is protected by a 3-step protocol enforced via hooks:

1. **Lookup** — `qbMasterData` to resolve vendor/customer and account IDs
2. **Duplicate check** — `qbFetchTransactions` to verify no duplicate exists
3. **Confirm** — Explicit user confirmation before writing

Additional guards:
- Void operations require fetching transaction details first
- Batch operations require master data lookup
- Bank feed flags must include `aiReasoning` for the CPA
- Journal entries get a debit = credit reminder

## Agent Memory

The plugin uses `agentMemory` to learn client-specific patterns over time:

| Type | Purpose | Example |
|------|---------|---------|
| `vendor` | Vendor-to-account mappings with confidence (upvotes) | "Office Depot → Office Supplies (6 upvotes)" |
| `customer` | Customer-to-income account mappings | "Client ABC → Consulting Income" |
| `client` | Client preferences and special rules | "All Uber rides go to Travel" |
| `worklog` | Autonomous loop state for crash recovery | Cycle count, status, last timestamp |

Memory naming: `{type}_{clientName}_{entityName}` (e.g., `vendor_acme_office-depot`)

### Bootstrap (First-Time Learning)

When a new client connects QuickBooks, `/bootstrap` reads the last 12 months of transactions and seeds memory automatically:
- Extracts vendor → account, customer → income, recurring patterns, transfer routes
- Caps every mapping at **5 upvotes** — the agent must earn higher trust through real-time accuracy
- Shows the full summary to the CPA for review before activating
- Tags all bootstrap memories with `source: "bootstrap"` for traceability

### Confidence Lifecycle
```
Connect QB → /bootstrap (cap 5) → Real usage upvotes (+1 each) → CPA corrections override → Confidence grows
```

The upvote system drives confidence-based decisions:
- **5+ upvotes**: Auto-categorize (high confidence)
- **3-4 upvotes**: Auto-categorize with note (medium)
- **1-2 upvotes**: Proceed with caution (low)
- **0 upvotes**: Flag for CPA review (new/unknown)

## Architecture

```
Plugin (this repo)
  ├── agents/     → Accountant + CFO personas
  ├── skills/     → Bookkeeping + Financial Analysis expertise
  ├── commands/   → 15 user-facing slash commands
  └── hooks/      → Safety validation guards
  ↓ HTTP Streamable
MCP Server (deepledger-mcp on Render)
  ├── 20+ QuickBooks tools
  ├── Agent infrastructure (work queue, memory, documents, bank feed)
  └── Supabase PostgreSQL backend
```

## Connecting QuickBooks

1. Log in to [deepledger.ai](https://deepledger.ai)
2. Navigate to Settings > QuickBooks
3. Click "Connect QuickBooks" and authorize access
4. Once connected, the plugin can read and write to your QB company

## Version

See [CHANGELOG.md](CHANGELOG.md) for release history.
