# Financial Analysis Skill

Expertise in generating financial reports, analyzing business health, computing ratios, identifying trends, and providing actionable CFO-level insights.

## Trigger

Activate this skill when the user wants to:
- Generate a P&L, Balance Sheet, Cash Flow, or aging report
- Analyze profitability, margins, or revenue trends
- Calculate financial ratios (current ratio, DSO, debt-to-equity, etc.)
- Assess cash runway or cash flow health
- Compare performance across periods
- Check customer concentration or vendor spending patterns
- Run a financial health check

## Report Types Available

| Report | Tool Call | Purpose |
|--------|-----------|---------|
| Profit & Loss | `qbReports(reportType="ProfitAndLoss")` | Revenue, expenses, net income |
| P&L Detail | `qbReports(reportType="ProfitAndLossDetail")` | Transaction-level drill-down |
| Balance Sheet | `qbReports(reportType="BalanceSheet")` | Assets, liabilities, equity |
| Cash Flow | `qbReports(reportType="CashFlow")` | Operating, investing, financing flows |
| Aged Receivables | `qbReports(reportType="AgedReceivables")` | Who owes you, how overdue |
| Aged Payables | `qbReports(reportType="AgedPayables")` | What you owe, when due |
| Trial Balance | `qbReports(reportType="TrialBalance")` | All accounts, debits = credits |
| Sales by Customer | `qbReports(reportType="SalesByCustomer")` | Revenue by customer |
| Customer Income | `qbReports(reportType="CustomerIncome")` | Customer profitability |
| Sales by Product | `qbReports(reportType="SalesByProduct")` | Product/service revenue |
| Vendor Expenses | `qbReports(reportType="VendorExpenses")` | Spending by vendor |

## Workflow: Full Financial Analysis

Follow the `getGuide(guideType="financial_analysis")` workflow:

### Step 1: Pull Core Financial Reports
```
qbReports → P&L (current month, QTD, YTD with prior period comparison)
qbReports → Balance Sheet (current date with prior month comparison)
qbReports → Cash Flow (current month and YTD)
```

Use `summarizeBy: "Month"` for trend visibility.
Keep `accountingMethod` consistent across all reports.

### Step 2: Analyze Profitability
Calculate from P&L data:
- **Gross Margin** = (Revenue - COGS) / Revenue
- **Operating Margin** = Operating Income / Revenue
- **Net Margin** = Net Income / Revenue

Drill deeper with:
- `SalesByCustomer` → revenue concentration risk
- `SalesByProduct` → which offerings are most profitable
- `VendorExpenses` → spending pattern analysis

### Step 3: Review Cash Position
From Cash Flow statement:
- Separate operating, investing, and financing flows
- **Cash Runway** = Cash on Hand / Average Monthly Cash Burn
- Review AR aging for incoming cash timing
- Review AP aging for upcoming obligations

Red flags:
- Operating cash flow negative while P&L shows profit → investigate AR
- Cash runway < 3 months → critical alert

### Step 4: Compute Key Ratios

**Liquidity:**
- Current Ratio = Current Assets / Current Liabilities (target > 1.5)
- Quick Ratio = (Current Assets - Inventory) / Current Liabilities

**Efficiency:**
- DSO = AR / (Annual Revenue / 365) — rising DSO = collection problems
- DPO = AP / (Annual COGS / 365)

**Leverage:**
- Debt-to-Equity = Total Liabilities / Equity

### Step 5: Identify Trends & Anomalies

Flag these conditions:
- Revenue declining 3+ consecutive months
- Expense categories growing faster than revenue
- Margin compression vs prior periods
- Top customer > 30% of revenue (concentration risk)
- DSO increasing month-over-month
- Cash runway < 3 months

Use `qbAccountHealth` to detect statistical outliers automatically.

### Step 6: Synthesize & Present

Structure the output:
1. **Headline** — One sentence: overall health
2. **Key Metrics Table** — Numbers with period-over-period comparison
3. **Trends** — Improving / declining / stable with percentages
4. **Risks** — Issues requiring attention
5. **Recommendations** — Specific, actionable next steps

Save key insights to `agentMemory` for longitudinal tracking.

## Presentation Rules

- Never present a number without comparison context
- Lead with the most impactful finding
- Use percentages for changes, not just absolute numbers
- Need 3+ data points to call something a "trend"
- Account for seasonality before flagging declines
- Never mix Accrual and Cash basis between reports
- Never compare a partial month to a full month

## Health Check Quick Mode

For a fast health check:
1. `qbAccountHealth` on all bank + CC accounts
2. `qbReports(reportType="ProfitAndLoss")` for current month
3. `qbReports(reportType="AgedReceivables")` for overdue AR
4. `qbReports(reportType="AgedPayables")` for upcoming AP

Present: health scores, headline P&L, overdue amounts, and any flags.
