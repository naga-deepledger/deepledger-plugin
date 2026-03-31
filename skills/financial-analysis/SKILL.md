---
name: Financial Analysis
description: >
  This skill should be used when the user asks to "show me the P&L",
  "how is my business doing", "what are my expenses", "revenue trends",
  "cash flow analysis", "compare this month to last month", "aging report",
  "who owes me money", "what bills are due", "financial health check",
  "variance analysis", "margin analysis", "cash runway", or any financial
  reporting and analysis task.
version: 2.0.0
---

## Purpose

Provide autonomous financial analysis expertise for interpreting QuickBooks
reports and surfacing actionable business insights. Operate proactively —
pull all needed data, analyze fully, and present complete findings without
asking intermediate questions.

## Operating Principle

**Never ask "would you like me to dig deeper?" — always dig deeper.**
Pull all relevant reports upfront, cross-reference between them, drill into
material variances automatically, and deliver complete analysis in a single
response. The user should get a full picture without having to prompt for
more detail.

## Analysis Framework

### Autonomous Report Selection

Based on the user's question, pull ALL relevant reports without asking:

| User Says | Pull These Reports |
|-----------|--------------------|
| "P&L" / "profit and loss" | ProfitAndLoss (current + prior) |
| "balance sheet" | BalanceSheet (current + prior) |
| "cash flow" / "cash runway" | CashFlow + BalanceSheet (for cash balance) |
| "aging" / "who owes" / "what's due" | AgedReceivables + AgedPayables |
| "how's my business" / "health check" | ProfitAndLoss + BalanceSheet + CashFlow + AgedReceivables + AgedPayables |
| "expenses" / "spending" | ProfitAndLoss + VendorExpenses |
| "revenue" / "income" / "sales" | ProfitAndLoss + CustomerIncome |
| Anything vague or general | ProfitAndLoss + BalanceSheet (minimum) |

### Always Compare Periods

Never present numbers in isolation. Always compare:
- Current month vs prior month
- Current quarter vs prior quarter
- Year-to-date vs prior year-to-date

Determine the right comparison automatically based on context.
Highlight variances with both % change and $ impact.

### Smart Period Detection

- No period specified → current month vs prior month
- "this quarter" → current quarter vs prior quarter + same quarter prior year
- "this year" / "YTD" → year-to-date vs prior year-to-date
- "how am I doing" → current month + YTD + trailing 3-month trend
- Specific month/quarter → that period vs prior year same period + preceding period

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

### Automatic Drill-Down

For ANY material variance, automatically drill into transaction-level data:
- Revenue or expense change >$5,000 → pull TransactionList for that account
- Any category change >20% → pull TransactionList for that account
- New account not in prior period with >$2,000 → investigate
- Account dropped to zero → note and explain if possible

Do NOT ask "would you like me to drill into this?" — just do it.

### Translate to Business Language

Convert accounting numbers into plain business insights:
- Not: "Revenue decreased 12% QoQ"
- Instead: "Revenue dropped $15K (12%) from last quarter — driven mainly
  by a $12K decline in consulting income. Worth investigating whether
  this is seasonal or if a key client reduced their engagement."

### Proactive Warning Signs

Check for and surface these on EVERY analysis, even if the user didn't ask:
- **Cash runway** — If current burn rate exceeds cash reserves
- **Margin compression** — Gross margin dropping >3% period-over-period
- **Aging receivables** — Invoices >60 days outstanding
- **Concentration risk** — Single customer >30% of revenue
- **Expense spikes** — Any category up >25% without obvious explanation
- **Consecutive losses** — Net losses for 2+ months
- **OpEx outpacing revenue** — Expenses growing faster than income
- **Overdue payables** — AP >60 days exceeding 30% of total
- **Declining cash** — Cash balance down 2+ consecutive months

Frame unsolicited warnings as: "While reviewing [X], I also noticed..."

### Back Up Insights with Data

For any major fluctuating account, fetch transaction-level detail using
qbFetchTransactions or qbReports (TransactionList) to identify the
specific transactions driving the variance. Always cite specifics.

## Output Format

1. **Headline insight** — the single most important finding (1-2 sentences)
2. **Key numbers table** — the critical metrics at a glance
3. **Detailed findings** — each major variance with context and transactions
4. **Warning signs** — any detected issues with severity and context
5. **Recommendations** — prioritized, actionable next steps

Bold key numbers and percentages. Use tables for data. Keep language
accessible to non-accountants.

## Additional Resources

### Reference Files

- **`references/variance-analysis.md`** — Detailed methodology for
  period-over-period comparison, materiality thresholds, and narrative
  templates
- **`references/warning-signs.md`** — Comprehensive list of financial
  red flags with detection logic and recommended actions
