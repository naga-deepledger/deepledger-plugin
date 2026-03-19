---
description: Cash flow statement and runway analysis
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period]
---

Generate a Cash Flow statement autonomously with runway analysis.

If $ARGUMENTS is provided, use it as the period. Default to current month.

Steps (execute ALL autonomously):
1. Use qbReports with "CashFlow" for the requested period
2. Use qbReports with "BalanceSheet" to get current cash balance
3. Break down by section:
   - Operating Activities (net income + adjustments)
   - Investing Activities (asset purchases/sales)
   - Financing Activities (loans, owner draws/contributions)
4. Calculate monthly burn rate from operating cash flow
5. Estimate cash runway: current cash balance / average monthly burn
6. Compare to prior period — is cash position improving or declining?
7. For any major cash flow item (>$5,000), drill into transactions
   automatically to explain what drove it
8. Check warning signs automatically:
   - Runway below 6 months → WARNING with urgency level
   - Runway below 3 months → CRITICAL with immediate action needed
   - Negative operating cash flow trend (2+ months)
   - Large investing outflows without corresponding revenue growth
   - Excessive owner draws relative to net income
9. Present: headline (cash runway number), cash flow summary table,
   burn rate, runway estimate, trend analysis, warning signs,
   recommendations
