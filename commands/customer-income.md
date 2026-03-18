---
description: Customer revenue analysis — who's paying you and how much
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period or customer name]
---

Analyze customer revenue. If $ARGUMENTS contains a customer name, focus on
that customer. If it contains a period, use it as the reporting period.
Default to current month.

Steps:
1. Use qbReports with reportType "CustomerIncome" for the requested period
2. Use qbReports again for the comparison period (prior month or prior year)
3. Rank customers by total revenue (highest first)
4. Calculate for each significant customer:
   - Total revenue this period
   - Change from prior period ($ and %)
   - Percentage of total revenue
5. Highlight:
   - Top 5 customers by revenue
   - Customers with >25% increase (growth opportunities)
   - Customers with >25% decrease (churn risk)
   - New customers not present in prior period
   - Lost customers (present last period, zero this period)
6. If a specific customer is requested:
   - Pull detailed transactions via qbFetchTransactions for that customer
   - Show individual invoices/payments with dates and amounts
   - Calculate average invoice size and payment timing
   - Check aging: any outstanding invoices >30 days?
7. Concentration risk analysis:
   - Flag any customer >30% of total revenue
   - Show top 3 customers as % of total
   - Compare concentration to prior period
8. Present: ranked customer table, growth/churn highlights, concentration warning
