# /balance-sheet

Generate a Balance Sheet report.

## Usage
```
/balance-sheet                 # As of today
/balance-sheet 3/31/2026       # As of specific date
/balance-sheet compare         # Current vs prior month
```

## Behavior

Use the **CFO** agent with the **Financial Analysis** skill.

1. Determine the as-of date (default: today)
2. Pull the report: `qbReports(reportType="BalanceSheet")` with date range
3. For `compare` mode, pull current and prior period side-by-side
4. Present the three sections:
   - **Assets** — Current Assets, Fixed Assets, Other Assets
   - **Liabilities** — Current Liabilities, Long-term Liabilities
   - **Equity** — Owner's Equity, Retained Earnings
5. Verify: Total Assets = Total Liabilities + Total Equity
6. Calculate key ratios:
   - **Current Ratio** = Current Assets / Current Liabilities (healthy > 1.5)
   - **Quick Ratio** = (Current Assets - Inventory) / Current Liabilities
   - **Debt-to-Equity** = Total Liabilities / Equity
7. Flag any concerns (Current Ratio < 1.0, negative equity, etc.)
