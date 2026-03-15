---
description: Cash flow statement and runway analysis
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period]
---

Generate a Cash Flow statement. If $ARGUMENTS is provided, use it as
the period. Default to current month.

Steps:
1. Use qbReports with reportType "CashFlow" for the requested period
2. Break down by section:
   - Operating Activities (net income + adjustments)
   - Investing Activities (asset purchases/sales)
   - Financing Activities (loans, owner draws/contributions)
3. Calculate monthly burn rate from operating cash flow
4. Get current cash balance from qbReports BalanceSheet
5. Estimate cash runway: current cash balance / average monthly burn
6. Compare to prior period — is cash position improving or declining?
7. Flag if runway is below 6 months
8. Present: cash flow summary, burn rate, runway estimate, trend
