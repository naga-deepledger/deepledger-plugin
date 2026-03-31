---
description: AR/AP aging report — who owes you, what you owe
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [receivables|payables|both]
---

Generate aging report autonomously. Default to both AR and AP.

Steps (execute ALL autonomously):
1. Determine type from $ARGUMENTS:
   - "receivables", "ar", "who owes" → AgedReceivables only
   - "payables", "ap", "what we owe" → AgedPayables only
   - anything else or empty → both
2. Use qbReports with "AgedReceivables" and/or "AgedPayables"
3. Summarize by aging bucket:
   - Current (not yet due)
   - 1-30 days past due
   - 31-60 days past due
   - 61-90 days past due
   - 90+ days past due
4. List top overdue items with customer/vendor name and amount
5. Calculate automatically:
   - Total outstanding
   - Average days outstanding
   - Percentage in each bucket
   - Concentration (any single entity >30% of total)
6. For items >60 days, pull specific invoice/bill details via
   qbFetchTransactions to show invoice numbers, dates, and amounts
7. Check warning signs automatically:
   - AR >60 days exceeds 20% of total → flag for collections
   - AP >60 days exceeds 30% of total → flag for vendor relations risk
   - Any single entity >50% of outstanding → concentration risk
   - Growing total outstanding vs prior month → cash flow concern
8. Present: headline (total outstanding + key concern), aging summary
   table, overdue highlights with specific items, action items
   prioritized by amount and age
