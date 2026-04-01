# /pnl

Generate a Profit & Loss (Income Statement) report.

## Usage
```
/pnl                          # Current month P&L
/pnl ytd                      # Year-to-date P&L
/pnl march 2026               # Specific month
/pnl q1 2026                  # Quarterly
/pnl compare                  # Current month vs prior month vs same month last year
```

## Behavior

Use the **CFO** agent with the **Financial Analysis** skill.

1. Determine the reporting period from the user's input (default: current month)
2. Pull the P&L report: `qbReports(reportType="ProfitAndLoss")` with appropriate date range
3. For `compare` mode, pull multiple periods and show side-by-side
4. Calculate and present:
   - **Revenue** — total and by category
   - **COGS** — cost of goods sold
   - **Gross Margin** — (Revenue - COGS) / Revenue as percentage
   - **Operating Expenses** — broken down by category
   - **Operating Margin** — Operating Income / Revenue
   - **Net Income** — bottom line
   - **Net Margin** — Net Income / Revenue
5. Highlight significant variances (> 10% change from prior period)
6. Provide 2-3 key insights and any recommended actions

## Options
- `summarizeBy: "Month"` for monthly trend breakdowns
- Use consistent `accountingMethod` (Accrual unless client prefers Cash)
- Always include at least one comparison period for context
