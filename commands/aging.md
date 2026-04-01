# /aging

Generate Aged Receivables and/or Aged Payables reports.

## Usage
```
/aging                         # Both AR and AP
/aging ar                      # Aged Receivables only
/aging ap                      # Aged Payables only
/aging ar overdue              # Only overdue receivables
```

## Behavior

Use the **CFO** agent with the **Financial Analysis** skill.

1. Determine which reports to pull (default: both AR and AP)
2. Pull reports:
   - `qbReports(reportType="AgedReceivables")` — who owes you
   - `qbReports(reportType="AgedPayables")` — what you owe
3. Present in aging buckets: Current, 1-30, 31-60, 61-90, 90+ days
4. Calculate:
   - **Total Outstanding AR** and **Total Outstanding AP**
   - **DSO** (Days Sales Outstanding) = AR / Daily Revenue
   - **DPO** (Days Payable Outstanding) = AP / Daily COGS
5. Highlight:
   - AR over 90 days → may need bad debt write-off or collections follow-up
   - AP past due → may have late payment penalties
   - Top 5 largest outstanding balances in each category
6. Recommendations:
   - Customers to follow up with for collection
   - Bills to prioritize for payment
   - Potential bad debt write-offs to discuss with CPA
