# Changelog

## [1.2.0] - 2026-04-01

### Added
- **`/bootstrap` command**: First-time client onboarding — reads 12 months of QB history, extracts vendor/customer/account mappings, presents summary to CPA for review, seeds agent memory with upvote cap of 5. Supports review, status, and reset modes.
- **Bootstrap workflow in bookkeeping skill**: Full extract → analyze → present → seed → mark workflow
- **Bootstrap detection in `/loop` and `/bank-feed`**: Auto-warns if client hasn't been bootstrapped, recommends running `/bootstrap` first
- **Amount range anomaly detection**: Bootstrap stores min/max/avg per vendor — flags transactions 3x outside learned range
- **Memory lifecycle documentation**: Bootstrap (cap 5) → real-time upvotes (+1) → CPA corrections override

### Changed
- **Accountant agent**: Added bootstrap as core responsibility #1, expanded memory section with lifecycle and bootstrap vs real-time distinction
- **README**: Added bootstrap to quick-start flow, command table, and agent memory section

## [1.1.0] - 2026-04-01

### Changed
- **Transport**: Switched from SSE to HTTP Streamable (`/sse` → `/mcp`) — the Anthropic-recommended transport for remote MCP servers

### Added
- **README**: Setup guide, quick-start, command reference, architecture overview, memory schema docs
- **`/budget` command**: Budget vs actuals comparison with variance analysis and budget creation
- **`/recurring` command**: List, create, pause, resume, and delete recurring transactions
- **Error recovery in `/loop`**: Crash resume via `lastCompletedStep` tracking, per-item failure isolation (log and continue), retry limits (3 attempts before flagging), worklog schema documentation
- **Batch operation workflow**: Step-by-step batch recording guide in bookkeeping skill with example payload and error handling
- **Corrections & reversals**: Void-and-rerecord vs reversing JE guidance with decision criteria in accountant agent and bookkeeping skill
- **Recurring transaction management**: Guidance in accountant agent for automated vs reminder types
- **Agent memory schema**: Documented types (vendor, customer, client, worklog), naming conventions, and upvote lifecycle in README

### Fixed
- Plugin was pointing at removed `/sse` endpoint — now uses the live `/mcp` endpoint

## [1.0.0] - 2026-03-21

### Added
- Initial release
- Accountant and CFO agents
- Bookkeeping and Financial Analysis skills
- 11 commands: /record, /bank-feed, /find, /transfer, /pnl, /balance-sheet, /cash-flow, /aging, /close-books, /health-check, /loop
- 5 safety hooks: write-safety-guard, void-transaction-guard, batch-safety-guard, bank-feed-flag-quality, journal-entry-balance-check
