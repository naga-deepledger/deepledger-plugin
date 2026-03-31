---
description: Cash flow statement, runway analysis, and 30/60/90-day forward projection
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period]
---

Generate a Cash Flow statement autonomously with runway analysis and forward cash projection.

If $ARGUMENTS is provided, use it as the period. Default to current month.

Steps (execute ALL autonomously):

**Phase 1: Historical Analysis**
1. Use qbReports "CashFlow" for the requested period
2. Use qbReports "CashFlow" for the prior period (for trend comparison)
3. Use qbReports "BalanceSheet" to get current cash balance (Bank + Other Current Asset accounts)
4. Break down by section:
   - Operating Activities (net income + adjustments)
   - Investing Activities (asset purchases/sales)
   - Financing Activities (loans, owner draws/contributions)
5. Calculate monthly burn rate from operating cash flow (average of current + prior period)
6. Estimate cash runway: current cash balance / average monthly burn
7. For any major cash flow item (>$5,000), drill into transactions via qbFetchTransactions
   to explain what drove it (do this autonomously — no asking)
8. Check historical warning signs:
   - Runway below 6 months → WARNING with urgency level
   - Runway below 3 months → CRITICAL with immediate action needed
   - Negative operating cash flow trend (2+ months)
   - Large investing outflows without corresponding revenue growth
   - Excessive owner draws relative to net income

**Phase 2: Forward Cash Projection (30/60/90 days)**

Pull forward-looking data in parallel:
- qbReports "AgedReceivables" (expected cash inflows — money customers owe us)
- qbReports "AgedPayables" (expected cash outflows — bills we owe vendors)
- qbRecurringTransaction(action: "list") to find predictable recurring flows

Build a 30/60/90-day cash projection:

```
Starting Cash Balance: [current cash from Balance Sheet]

--- 30-Day Projection ---
Expected Inflows:
  + Current AR (0-30 days overdue): [amount from AgedReceivables Current column]
  + Recurring income (subscriptions, retainers due in 30 days): [from RecurringTransactions]
Expected Outflows:
  - Bills due in 30 days (Current + 1-30 days): [amount from AgedPayables Current+1-30 columns]
  - Recurring expenses due in 30 days: [from RecurringTransactions]
  - Estimated operating burn: [monthly burn rate × 1 month]
Projected Cash @ Day 30: [Starting + Inflows - Outflows]

--- 60-Day Projection ---
(Carry forward Day 30 balance)
  + AR 31-60 days overdue (partially collectible — apply 70% collection rate)
  + Recurring income months 2
  - AP 31-60 days
  - Recurring expenses month 2
  - Operating burn month 2
Projected Cash @ Day 60: [...]

--- 90-Day Projection ---
(Carry forward Day 60 balance)
  + AR 61-90 days (apply 50% collection rate — more uncertain)
  + Recurring income month 3
  - AP 61-90 days
  - Recurring expenses month 3
  - Operating burn month 3
Projected Cash @ Day 90: [...]
```

Collection rate assumptions:
- Current AR (0-30 days): 95% likely to collect
- 31-60 days overdue AR: 70% likely to collect
- 61-90 days overdue AR: 50% likely to collect
- 91+ days overdue AR: 20% likely to collect (high risk)

Flag projection risks:
- If Day 30 projected balance < 1 month burn → CRITICAL cash warning
- If Day 60 projected balance < 2 months burn → WARNING: plan now
- If AR >60 days overdue represents >25% of total AR → collection risk note
- If any single customer accounts for >40% of outstanding AR → concentration risk

**Phase 3: Present Results**
9. Present in this order:
   - **Headline**: Cash runway + projected position in one sentence
   - **Current Position**: Cash balance, burn rate, historical runway months
   - **Cash Flow Summary Table** (Operating / Investing / Financing / Net Change)
   - **30/60/90-Day Forward Projection Table** with confidence note
   - **Collection Risk** (if any AR concerns)
   - **Warning Signs** with severity (CRITICAL / WARNING / WATCH)
   - **Recommendations**: Specific action items (collect from [Customer], delay [Bill], etc.)

Use plain language. Give specific numbers, not vague percentages.

**Phase 4: Save Cash Flow Forecast to Agent Memory**

After presenting results, save a cash flow forecast snapshot to agent memory so the
portal dashboard can display projected cash positions without re-running the command:

```
agentMemory(
  operation: "write" (or "update" if memoryId exists),
  category: "cash_flow_forecast",
  subject: "latest_forecast",
  memory: "<JSON string: {
    \"report_date\": \"<ISO date>\",
    \"period\": \"<e.g. 2026-03>\",
    \"current_cash\": <number>,
    \"burn_rate_monthly\": <number>,
    \"runway_months\": <number>,
    \"projected_cash_30d\": <number>,
    \"projected_cash_60d\": <number>,
    \"projected_cash_90d\": <number>,
    \"ar_total\": <number>,
    \"ap_total\": <number>,
    \"recurring_income_monthly\": <number>,
    \"recurring_expense_monthly\": <number>,
    \"risk_level\": \"healthy\" | \"watch\" | \"warning\" | \"critical\",
    \"top_risks\": [\"<risk description>\", ...],
    \"top_recommendations\": [\"<action item>\", ...]
  }>",
  confidence: 90
)
```

If a memory with category="cash_flow_forecast" and subject="latest_forecast" already
exists, use operation: "update" with the memoryId to overwrite it.
