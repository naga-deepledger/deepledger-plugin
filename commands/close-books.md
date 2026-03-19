---
description: Month-end close checklist and validation
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [month/period]
---

Run a month-end close checklist for the specified period: "$ARGUMENTS"

Default to the most recently completed month if not specified. This is a
structured review that validates the books are clean and ready to close.

Execute ALL steps autonomously — pull all data, analyze, and present a
complete close readiness report in one response.

Steps:

**Phase 1: Data Pull (do all in parallel where possible)**
1. qbReports "ProfitAndLoss" for the closing month
2. qbReports "BalanceSheet" as of month end
3. qbReports "TrialBalance" for the closing month
4. qbReports "AgedReceivables" as of month end
5. qbReports "AgedPayables" as of month end
6. qbReports "TransactionList" for the closing month (to scan for anomalies)

**Phase 2: Checklist Validation**

Check each item and report status (Pass / Warning / Action Needed):

| # | Check | What to Look For |
|---|-------|-----------------|
| 1 | **Trial Balance** | Debits = Credits? Any out-of-balance accounts? |
| 2 | **Undeposited Funds** | Balance should be $0 or near-zero at month end. Outstanding amounts indicate undeposited receipts. |
| 3 | **Uncategorized Income/Expense** | Any transactions in "Uncategorized" or "Ask My Accountant" accounts? These need reclassification. |
| 4 | **Suspense Accounts** | Any balance in suspense or clearing accounts that should be zero? |
| 5 | **AP Accuracy** | Do open bills match what's actually owed? Any bills >90 days that should be voided or paid? |
| 6 | **AR Accuracy** | Do open invoices match what's expected? Any invoices >90 days that should be written off? |
| 7 | **Bank Reconciliation** | Remind user to reconcile bank/credit card statements (can't be done programmatically) |
| 8 | **Recurring Transactions** | Were all expected recurring transactions recorded? (Check for missing rent, subscriptions, payroll) |
| 9 | **Large or Unusual Entries** | Any single transaction >2x the average for its account? Any transactions on weekends/holidays? |
| 10 | **Revenue Recognition** | Is revenue recorded in the correct period? Any invoices dated in this month that belong to next? |
| 11 | **Prepaid Expenses** | Any prepaid expense accounts that need amortization entries? |
| 12 | **Depreciation** | Were depreciation entries recorded for fixed assets? |
| 13 | **Accruals** | Any known expenses incurred but not yet billed (accrued liabilities)? |
| 14 | **Payroll** | Were all payroll entries recorded and categorized correctly? |
| 15 | **Sales Tax** | Any sales tax collected that needs to be remitted? |

**Phase 3: Results Presentation**

Present as a structured report:

1. **Close Readiness Score**: X/15 checks passed
2. **Summary**: One-line assessment ("Books are clean" or "3 items need attention")
3. **Checklist Table**: Each item with status icon and finding
4. **Action Items**: Specific transactions that need attention, with:
   - What needs to be done (reclassify, void, create JE, etc.)
   - Transaction details (date, amount, vendor, current account)
   - Suggested correction
5. **Proposed Journal Entries**: If adjustments are needed (depreciation,
   accruals, reclassifications), propose them — ready for user approval
6. **Reminder**: Items that can't be verified programmatically (bank reconciliation,
   physical inventory counts, etc.)

**Tone:** Professional but accessible. Frame findings as "here's what I found
and here's how to fix it" — not just a list of problems.
