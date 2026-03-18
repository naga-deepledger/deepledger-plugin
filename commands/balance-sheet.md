---
description: Balance sheet snapshot with key ratios
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [date]
---

Generate a Balance Sheet report autonomously. Pull all data, calculate ratios,
and present complete findings.

If $ARGUMENTS is provided, use it as the as-of date. Default to today.

Steps (execute ALL autonomously):
1. Use qbReports with "BalanceSheet" for the requested date
2. Use qbReports again for comparison (prior month-end or prior quarter-end)
3. Summarize by major section:
   - Current Assets (cash, AR, inventory)
   - Fixed Assets
   - Current Liabilities (AP, credit cards, short-term debt)
   - Long-term Liabilities
   - Equity
4. Calculate key ratios automatically:
   - Current ratio (current assets / current liabilities)
   - Quick ratio ((current assets - inventory) / current liabilities)
   - Debt-to-equity (total liabilities / total equity)
5. Compare to prior period and highlight significant changes
6. For any major balance change (>20% or >$10K), drill into transactions
   automatically to explain what drove the change
7. Check warning signs automatically:
   - Negative working capital (current liabilities > current assets)
   - High debt ratio (debt-to-equity > 2.0)
   - Declining cash (check last 3 months trend)
   - Growing AR faster than revenue
   - Overdue AP (cross-reference with aging if significant)
8. Present: headline finding, summary table, ratios with interpretation,
   period comparison, warning signs, recommendations
