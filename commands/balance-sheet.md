---
description: Balance sheet snapshot with key ratios
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [date]
---

Generate a Balance Sheet report. If $ARGUMENTS is provided, use it as
the as-of date. Default to today.

Steps:
1. Use qbReports with reportType "BalanceSheet" for the requested date
2. Use qbReports again for the comparison date (prior month-end or prior quarter-end)
3. Summarize by major section:
   - Current Assets (cash, AR, inventory)
   - Fixed Assets
   - Current Liabilities (AP, credit cards, short-term debt)
   - Long-term Liabilities
   - Equity
4. Calculate key ratios:
   - Current ratio (current assets / current liabilities)
   - Quick ratio ((current assets - inventory) / current liabilities)
   - Debt-to-equity (total liabilities / total equity)
5. Compare to prior period and highlight significant changes
6. Flag concerns: negative working capital, high debt ratio, declining cash
7. Present: summary table, ratios, period comparison, narrative
