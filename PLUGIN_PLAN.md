# DeepLedger Claude Code Plugin — Implementation Plan (SUPERSEDED)

> **Status**: ⛔ SUPERSEDED — do not use. Removed 2026-07-23 during the documentation
> conflict sweep. This was the pre-1.0 draft plan (2026-03-14) and contradicted the
> shipped v2.0.0 plugin on nearly every axis:
>
> - **Safety model**: the draft required explicit user confirmation before every write
>   ("Analyze → Propose → Confirm → Execute"); the shipped model is the decide gate
>   (user-requested / CPA-approved / history-consistent, else escalate via `tasks`) with
>   confirmation reserved for interrupts.
> - **Components**: the draft specified 6 slash commands and 2 agents; v2.0.0 ships
>   10 skills + hooks + the MCP connector, with no commands or agents.
> - **Tool roster**: the draft listed 15 QB tools and no platform tools; the server
>   exposes 23 tools (17 QuickBooks + 6 platform).
> - **Categorization**: the draft said "follow the last 3-5 transactions, ask when
>   uncertain"; the shipped rule is the quantified consistency rule (charter §6).
>
> **Current sources of truth**: `DEEPLEDGER-PHILOSOPHY.md` and
> `deepledger-mcp-charter.md` at the project root, plus this repo's `README.md`.
> The original draft remains available in git history (this file, prior to 2026-07-23).
