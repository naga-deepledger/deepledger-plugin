---
name: Financial Analysis
description: >
  This skill should be used when the user asks to "show me the P&L",
  "how is my business doing", "what are my expenses", "revenue trends",
  "cash flow analysis", "compare this month to last month", "aging report",
  "who owes me money", "what bills are due", "financial health check",
  "variance analysis", "margin analysis", "cash runway", or any financial
  reporting and analysis task.
version: 1.0.0
---

## Purpose

Provide financial analysis expertise for interpreting QuickBooks reports
and surfacing actionable business insights.

## Analysis Framework

### Always Compare Periods

Never present numbers in isolation. Always compare:
- Current month vs prior month
- Current quarter vs prior quarter
- Year-to-date vs prior year-to-date

Highlight variances with both % change and $ impact.

### Report Types

| Report | Tool Call | Use When |
|--------|-----------|----------|
| Profit & Loss | qbReports (ProfitAndLoss) | Revenue, expenses, net income |
| Balance Sheet | qbReports (BalanceSheet) | Assets, liabilities, equity |
| Cash Flow | qbReports (CashFlow) | Cash position and movement |
| AR Aging | qbReports (AgedReceivables) | Who owes money |
| AP Aging | qbReports (AgedPayables) | Bills due |
| Trial Balance | qbReports (TrialBalance) | Account balances |
| Transaction List | qbReports (TransactionList) | Detailed transactions |
| Customer Income | qbReports (CustomerIncome) | Revenue by customer |
| Vendor Expenses | qbReports (VendorExpenses) | Spend by vendor |

### Translate to Business Language

Convert accounting numbers into plain business insights:
- Not: "Revenue decreased 12% QoQ"
- Instead: "Revenue dropped $15K (12%) from last quarter — driven mainly
  by a $12K decline in consulting income. Worth investigating whether
  this is seasonal or if a key client reduced their engagement."

### Proactive Warning Signs

Surface these automatically when detected:
- **Cash runway** — If current burn rate exceeds cash reserves
- **Margin compression** — Gross margin dropping >3% period-over-period
- **Aging receivables** — Invoices >60 days outstanding
- **Concentration risk** — Single customer >30% of revenue
- **Expense spikes** — Any category up >25% without obvious explanation

### Back Up Insights with Data

For any major fluctuating account, fetch transaction-level detail using
qbFetchTransactions or qbReports (TransactionList) to identify the
specific transactions driving the variance.

## Additional Resources

### Reference Files

- **`references/variance-analysis.md`** — Detailed methodology for
  period-over-period comparison, materiality thresholds, and narrative
  templates
- **`references/warning-signs.md`** — Comprehensive list of financial
  red flags with detection logic and recommended actions
