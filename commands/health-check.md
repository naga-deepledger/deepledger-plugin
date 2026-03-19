---
description: Comprehensive financial health assessment with warning signs
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period]
---

Run a comprehensive financial health check autonomously. Pull ALL data
upfront, analyze everything, and present complete findings in one response.

If $ARGUMENTS is provided, use it as the period. Default to current month
vs prior month.

Steps (execute ALL autonomously — no intermediate questions):
1. Pull ALL reports needed for a full picture:
   - qbReports "ProfitAndLoss" for current AND prior period
   - qbReports "BalanceSheet" for current date
   - qbReports "CashFlow" for current period
   - qbReports "AgedReceivables"
   - qbReports "AgedPayables"
   - qbReports "CustomerIncome" for concentration check
2. Calculate key health metrics:
   - **Cash runway**: current cash / average monthly burn rate
   - **Gross margin %** and trend vs prior period
   - **Net income** and trend
   - **Current ratio**: current assets / current liabilities
   - **Days Sales Outstanding (DSO)**: (AR / Revenue) × days in period
   - **Revenue growth rate**: current vs prior period
   - **OpEx ratio**: operating expenses / revenue
3. Check for ALL warning signs:
   - Cash runway below 6 months
   - Gross margin compression >3 percentage points
   - Net losses (especially 2+ consecutive months)
   - Operating expenses growing faster than revenue
   - AR aging >60 days exceeding 20% of total AR
   - Single customer >30% of revenue
   - AP >60 days exceeding 30% of total AP
   - Any expense category up >25% and >$2,000
   - Declining cash balance for 2+ consecutive months
4. For ANY warning sign detected, automatically drill into transaction-level
   data to identify root causes — don't ask, just investigate
5. Present results:
   - **Headline**: Overall health in one sentence
   - **Health Scorecard**: Table of metrics with status (Healthy/Warning/Critical)
   - **Warning Signs**: Each issue with context, severity, and root cause
   - **Bright Spots**: Positive trends worth noting
   - **Key Numbers**: Cash balance, monthly revenue, monthly burn, net income
   - **Recommendations**: Prioritized action items (most urgent first)
6. Use plain business language — no unexplained jargon
