---
description: Vendor spend analysis — who you're paying and how much
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period or vendor name]
---

Analyze vendor spending autonomously. Pull all data and present complete
analysis without intermediate questions.

If $ARGUMENTS contains a vendor name, focus on that vendor. If it contains
a period, use it as the reporting period. Default to current month.

Steps (execute ALL autonomously):
1. Use qbReports with "VendorExpenses" for the requested period
2. Use qbReports again for the comparison period (prior month or prior year)
3. Rank vendors by total spend (highest first)
4. Calculate for each significant vendor:
   - Total spend this period
   - Change from prior period ($ and %)
   - Percentage of total expenses
5. Automatically highlight:
   - Top 5 vendors by spend
   - Vendors with >25% increase from prior period
   - New vendors not present in prior period
   - Vendors with significant decrease (possible missed payments?)
   - Any single vendor >20% of total expenses (concentration risk)
6. If a specific vendor is requested:
   - Pull detailed transactions via qbFetchTransactions for that vendor
   - Show individual transactions with dates, amounts, and accounts
   - Identify spending patterns (recurring vs one-time)
   - Show account categorization breakdown
7. Present: headline finding, ranked vendor table, variance highlights,
   concentration analysis, recommendations
