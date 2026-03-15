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
