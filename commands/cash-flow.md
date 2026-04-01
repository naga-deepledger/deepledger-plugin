# /cash-flow

Generate a Cash Flow Statement report.

## Usage
```
/cash-flow                     # Current month
/cash-flow ytd                 # Year-to-date
/cash-flow q1 2026             # Quarterly
```

## Behavior

Use the **CFO** agent with the **Financial Analysis** skill.

1. Determine the reporting period (default: current month)
2. Pull the report: `qbReports(reportType="CashFlow")` with date range
3. Present the three sections:
   - **Operating Activities** — Cash from core business operations
   - **Investing Activities** — Asset purchases/sales
   - **Financing Activities** — Loans, equity, distributions
4. Calculate:
   - **Net Cash Change** for the period
   - **Cash Runway** = Cash on Hand / Average Monthly Cash Burn
5. Cross-reference:
   - Pull `AgedReceivables` to estimate incoming cash timing
   - Pull `AgedPayables` to understand upcoming obligations
6. Flag:
   - Operating cash flow negative while P&L shows profit → investigate AR collections
   - Cash runway < 3 months → critical alert
   - Net cash change doesn't reconcile with Balance Sheet cash change → investigate
