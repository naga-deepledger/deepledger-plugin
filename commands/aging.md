---
description: AR/AP aging report — who owes you, what you owe
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [receivables|payables|both]
---

Generate aging report. Default to both AR and AP if $ARGUMENTS is empty.

Steps:
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
5. Calculate:
   - Total outstanding
   - Average days outstanding
   - Percentage in each bucket
6. Flag items >60 days as needing follow-up
7. For large overdue amounts, note the specific invoices/bills
8. Present: aging summary table, overdue highlights, action items
