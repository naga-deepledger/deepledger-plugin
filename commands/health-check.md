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

   **Profitability Metrics (from ProfitAndLoss)**:
   - **Gross margin %**: (Total Income - COGS) / Total Income × 100
   - **Net margin %**: Net Income / Total Income × 100
   - **Net income** and trend vs prior period
   - **Revenue growth rate**: (current revenue - prior revenue) / prior revenue × 100
   - **OpEx ratio**: operating expenses / revenue × 100

   **Liquidity Metrics (from BalanceSheet)**:
   - **Current ratio**: Total Current Assets / Total Current Liabilities
     (Healthy: >2.0, Warning: 1.0-2.0, Critical: <1.0)
   - **Quick ratio**: (Current Assets - Inventory) / Current Liabilities
     (Healthy: >1.0, Warning: 0.5-1.0, Critical: <0.5)
     Note: Pull inventory from BalanceSheet "Inventory" or "Inventory Asset" line.
     If no inventory account exists, quick ratio = current ratio.
   - **Working capital**: Current Assets - Current Liabilities (absolute $)

   **Solvency Metrics (from BalanceSheet)**:
   - **Debt-to-equity ratio**: Total Liabilities / Total Equity
     (Healthy: <2.0, Warning: 2.0-4.0, Critical: >4.0)
     Note: If equity is zero or negative, flag as CRITICAL — business is
     technically insolvent.

   **Efficiency Metrics (from ProfitAndLoss + Aging)**:
   - **Days Sales Outstanding (DSO)**: (AR / Revenue) × days in period
     (Healthy: <45, Warning: 45-60, Critical: >60)
   - **Days Payable Outstanding (DPO)**: (AP / COGS) × days in period
     (Healthy: 30-60, Warning: >60 or <15)

   **Cash Metrics (from CashFlow + BalanceSheet)**:
   - **Cash runway**: current cash / average monthly burn rate
     (Healthy: >6 months, Warning: 3-6, Critical: <3)
   - **Monthly burn rate**: average of current + prior period operating cash outflow
   - **Cash as % of assets**: Cash / Total Assets × 100
3. Check for ALL warning signs:
   - Cash runway below 6 months (CRITICAL if <3 months)
   - Gross margin compression >3 percentage points
   - Net margin negative (operating at a loss)
   - Net losses (especially 2+ consecutive months)
   - Operating expenses growing faster than revenue
   - Current ratio below 1.5 (WARNING), below 1.0 (CRITICAL)
   - Quick ratio below 1.0 (WARNING), below 0.5 (CRITICAL)
   - Debt-to-equity above 3.0 (WARNING), above 5.0 (CRITICAL)
   - Negative or zero equity (CRITICAL — technical insolvency)
   - Negative working capital (CRITICAL)
   - AR aging >60 days exceeding 20% of total AR
   - DSO above 45 days (WARNING), above 60 (CRITICAL)
   - Single customer >30% of revenue
   - AP >60 days exceeding 30% of total AP
   - Any expense category up >25% and >$2,000
   - Declining cash balance for 2+ consecutive months
   - Cash as % of total assets below 5% (liquidity concern)
4. For ANY warning sign detected, automatically drill into transaction-level
   data to identify root causes — don't ask, just investigate
5. **Save financial KPI snapshot** for the portal dashboard:
   After computing all metrics, call `agentMemory` to store a snapshot:
   - category: "financial_snapshot"
   - subject: "latest_kpis"
   - action: "remember"
   - memory: JSON object with these fields:
     ```json
     {
       "revenue": <total income from PnL, number>,
       "expenses": <total expenses from PnL, number>,
       "net_income": <net income from PnL, number>,
       "cash_on_hand": <cash/bank total from BalanceSheet, number>,
       "ar_outstanding": <total AR from AgedReceivables, number>,
       "ap_outstanding": <total AP from AgedPayables, number>,
       "current_ratio": <calculated current ratio, number>,
       "gross_margin_pct": <gross margin percentage, number>,
       "net_margin_pct": <net margin percentage, number>,
       "burn_rate_monthly": <average monthly operating cash outflow, number>,
       "runway_months": <cash / monthly burn, number>,
       "report_date": "<today's date YYYY-MM-DD>",
       "period": "<report period description e.g. 'Jan-Mar 2026'>"
     }
     ```
   This enables the portal dashboard to display live financial KPI cards
   without requiring users to manually pull reports.
6. Present results:
   - **Headline**: Overall health in one sentence
   - **Health Scorecard**: Table of ALL metrics with status (Healthy/Warning/Critical):

     | Metric | Value | Status | Benchmark |
     |--------|-------|--------|-----------|
     | Current Ratio | X.X | Healthy/Warning/Critical | >2.0 |
     | Quick Ratio | X.X | Healthy/Warning/Critical | >1.0 |
     | Debt-to-Equity | X.X | Healthy/Warning/Critical | <2.0 |
     | Working Capital | $X,XXX | Positive/Negative | Positive |
     | Gross Margin | XX% | Healthy/Compressing | >50% (SaaS) / >30% (services) |
     | Net Margin | XX% | Profitable/Loss | >10% |
     | DSO | XX days | Healthy/Warning/Critical | <45 days |
     | DPO | XX days | Healthy/Warning | 30-60 days |
     | Cash Runway | XX months | Healthy/Warning/Critical | >6 months |
     | Revenue Growth | XX% | Growing/Flat/Declining | >0% |
     | OpEx Ratio | XX% | Healthy/High | <80% |
     | Cash % of Assets | XX% | Healthy/Low | >5% |

   - **Warning Signs**: Each issue with context, severity, and root cause
   - **Bright Spots**: Positive trends worth noting
   - **Key Numbers**: Cash balance, working capital, monthly revenue, monthly burn, net income
   - **Recommendations**: Prioritized action items (most urgent first)
7. Use plain business language — no unexplained jargon
