---
description: Generate P&L report with variance analysis
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period]
---

Generate a Profit & Loss report autonomously. Pull all data, analyze fully,
and present complete findings — no intermediate questions.

If $ARGUMENTS is provided, use it as the reporting period. If no period
specified, default to current month vs prior month.

Steps (execute ALL autonomously):
1. Determine period from $ARGUMENTS or default to current month
2. Use qbReports with "ProfitAndLoss" for BOTH the requested period AND
   the comparison period (prior month or prior year same period) — pull both
3. Calculate variances (% change and $ impact) for each major category:
   - Revenue / Income
   - Cost of Goods Sold
   - Gross Profit (and gross margin %)
   - Operating Expenses (by category)
   - Net Income
4. For ANY significant variance (>$5,000 or >20%), AUTOMATICALLY drill into
   transactions using qbFetchTransactions or qbReports (TransactionList) —
   do NOT ask if the user wants to see details, just pull them
5. Translate findings into plain business language — not just numbers but
   what they mean and what's driving them
6. Check for ALL warning signs automatically:
   - Gross margin compression >3%
   - Any expense category up >25%
   - Revenue concentration (pull CustomerIncome if revenue changed significantly)
   - Net losses (especially consecutive)
   - OpEx growing faster than revenue
7. Present in this format:
   - **Headline**: The single most important finding (1-2 sentences)
   - **Summary table**: Key P&L lines with current, prior, $ change, % change
   - **Major variances**: Each material change with transaction-level explanation
   - **Warning signs**: Any detected issues
   - **Recommendations**: Prioritized action items
