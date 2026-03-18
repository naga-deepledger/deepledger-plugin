---
description: Comprehensive financial health assessment with warning signs
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period]
---

Run a comprehensive financial health check. If $ARGUMENTS is provided, use it
as the period. Default to current month vs prior month.

Steps:
1. Pull multiple reports in sequence for a full picture:
   - qbReports with "ProfitAndLoss" for current and prior period
   - qbReports with "BalanceSheet" for current date
   - qbReports with "CashFlow" for current period
   - qbReports with "AgedReceivables"
   - qbReports with "AgedPayables"
2. Calculate key health metrics:
   - **Cash runway**: current cash / average monthly burn rate
   - **Gross margin %** and trend vs prior period
   - **Net income** and trend
   - **Current ratio**: current assets / current liabilities
   - **Days Sales Outstanding (DSO)**: (AR / Revenue) × days in period
   - **Revenue growth rate**: current vs prior period
3. Check for warning signs (surface ALL that apply):
   - Cash runway below 6 months
   - Gross margin compression >3 percentage points
   - Net losses (especially 2+ consecutive months)
   - Operating expenses growing faster than revenue
   - AR aging >60 days exceeding 20% of total AR
   - Single customer >30% of revenue (pull CustomerIncome to check)
   - AP >60 days exceeding 30% of total AP
   - Any expense category up >25% and >$2,000
4. For any warning sign detected, drill into transaction-level data via
   qbFetchTransactions or qbReports (TransactionList) to identify root causes
5. Present results in this format:
   - **Health Score Summary**: Table of metrics with status indicators
   - **Warning Signs**: Each detected issue with context and severity
   - **Bright Spots**: Positive trends worth noting
   - **Recommendations**: Prioritized action items (most urgent first)
   - **Key Numbers**: Cash balance, monthly revenue, monthly burn, net income
6. Use plain business language throughout — no unexplained jargon
