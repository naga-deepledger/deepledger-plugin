---
name: month-end-close
description: Execute the month-end close workflow — reconcile accounts, review AP/AR, record accruals and depreciation, review financial statements, and run the 16-point checklist. Use when the user mentions closing the books, month-end, close period, or adjusting entries.
---

# Month-End Close Skill

Run the structured month-end close process: reconcile all accounts, clear outstanding items, record adjusting entries, review financial statements, and verify the books are ready for CPA review.

## Trigger

Activate when the user wants to:
- Close the books for a month
- Record accruals, deferrals, or depreciation
- Run the month-end checklist
- Check close status for a period
- Prepare financial statements for review

## The Close Process

Six steps, executed in order. Each step updates the portal in real-time via `closeRun`.

```
1. Reconcile All Accounts
2. Review AP & AR
3. Record Accruals & Deferrals
4. Record Depreciation
5. Review Financial Statements
6. Final Trial Balance
→ Score (0-16) → Status: needs_review → CPA approves in portal
```

## Before Starting

```
closeRun(operation="start", period="YYYY-MM", periodLabel="Month Year")
```

This creates the close run in the portal and sets status to `in_progress`.

## Step 1: Reconcile All Accounts

1. **Get accounts** — `qbMasterData(entityTypes=["account"])` → Bank and Credit Card types
2. **Health check** — `qbAccountHealth` on each account for the closing month
3. **Target** — All accounts >= 90 health score
4. **Resolve** — Fix duplicates, categorize uncategorized items, investigate outliers
5. **Update portal**:
   ```
   closeRun(operation="updateStep", step={id: "reconciliation", ...})
   closeRun(operation="updateChecks", checks=[{id: 7, label: "Bank Reconciliation", ...}])
   ```

## Step 2: Review AP & AR

1. **AP** — `qbReports(reportType="AgedPayables")` → verify all bills are in correct period
2. **AR** — `qbReports(reportType="AgedReceivables")` → verify all invoices are in correct period
3. **Flag** — Past-due AR over 90 days (potential bad debt write-off)
4. **Flag** — Past-due AP (late payment risk)
5. **Update portal**:
   ```
   closeRun(operation="updateStep", step={id: "ap_ar", ...})
   closeRun(operation="updateChecks", checks=[{id: 5, ...}, {id: 6, ...}])
   ```

## Step 3: Record Accruals & Deferrals

1. **Reverse prior month** — Check prior month JEs for accruals that need reversal
2. **New accruals** — Record via `qbJournalEntry`:
   - Accrued expenses: Debit Expense, Credit Accrued Liabilities
   - Prepaid amortization: Debit Expense, Credit Prepaid Asset
   - Deferred revenue: Debit Deferred Revenue, Credit Revenue
3. **Batch** — Use `qbBatch` for multiple JEs
4. **Status** — Proposed entries get `status: "proposed"` — CPA approves in portal before posting
5. **Update portal**:
   ```
   closeRun(operation="updateStep", step={id: "accruals", ...})
   closeRun(operation="addEntries", adjustingEntries=[...])
   closeRun(operation="updateChecks", checks=[{id: 11, ...}, {id: 13, ...}])
   ```

## Step 4: Record Depreciation

1. **Check history** — Review prior depreciation JEs for consistent monthly amounts
2. **Record** — `qbJournalEntry`: Debit Depreciation Expense, Credit Accumulated Depreciation
3. **Memo** — Include asset description (e.g., "Monthly depreciation — Office Equipment")
4. **Update portal**:
   ```
   closeRun(operation="updateStep", step={id: "depreciation", ...})
   closeRun(operation="addEntries", adjustingEntries=[...])
   closeRun(operation="updateChecks", checks=[{id: 12, ...}])
   ```

## Step 5: Review Financial Statements

1. **P&L** — `qbReports(reportType="ProfitAndLoss")` — compare to prior month and same month last year
2. **Balance Sheet** — `qbReports(reportType="BalanceSheet")` — verify Assets = Liabilities + Equity
3. **Cash Flow** — `qbReports(reportType="CashFlow")` — reconcile to cash change on Balance Sheet
4. **Flag** — Variances > 5% in gross margin vs prior periods
5. **Update portal**:
   ```
   closeRun(operation="updateStep", step={id: "statements", ...})
   closeRun(operation="setFinancials", financials={revenue, expenses, net_income, ...})
   closeRun(operation="updateChecks", checks=[{id: 9, ...}, {id: 10, ...}])
   ```

## Step 6: Final Trial Balance

1. **Pull** — `qbReports(reportType="TrialBalance")` — confirm total debits = total credits
2. **Investigate** — If out of balance, look for one-sided entries or posting errors
3. **Update remaining checks** — 1 (Trial Balance), 2 (Undeposited Funds), 3 (Uncategorized), 4 (Suspense), 8 (Recurring), 14 (Payroll), 15 (Sales Tax), 16 (Changes Since Last Close)
4. **Update portal**:
   ```
   closeRun(operation="updateStep", step={id: "trial_balance", ...})
   closeRun(operation="updateChecks", checks=[...remaining checks...])
   ```

## Final: Score and Complete

1. **Count** — checks with status "pass" → `score` (out of 16)
2. **Count** — checks with status "action_needed" → `actionItems`
3. **Complete** — `closeRun(operation="complete", score=N, actionItems=N)`
4. **Status** — Sets to `needs_review` — only a CPA can finalize in the portal

## 16-Point Checklist Reference

| # | Check | What to Verify |
|---|-------|----------------|
| 1 | Trial Balance | Debits = credits |
| 2 | Undeposited Funds | Balance at $0 or near-zero |
| 3 | Uncategorized Transactions | Nothing in Uncategorized/Ask My Accountant |
| 4 | Suspense Accounts | Clearing accounts at zero |
| 5 | AP Accuracy | Open bills match obligations |
| 6 | AR Accuracy | Open invoices match expectations |
| 7 | Bank Reconciliation | All bank/CC accounts reconciled |
| 8 | Recurring Transactions | All expected recurring items posted |
| 9 | Large/Unusual Entries | No unexplained amounts > 2x average |
| 10 | Revenue Recognition | Revenue in correct period |
| 11 | Prepaid Expenses | Amortization entries recorded |
| 12 | Depreciation | Fixed asset depreciation posted |
| 13 | Accruals | Known incurred-but-not-billed expenses accrued |
| 14 | Payroll | All entries recorded and categorized |
| 15 | Sales Tax | Collected tax accounted for |
| 16 | Changes Since Last Close | No unapproved edits to closed periods |

## Safety Checklist

- [ ] All accounts reconciled before proceeding to adjusting entries
- [ ] Prior month accruals reversed before recording new ones
- [ ] All adjusting entries default to `status: "proposed"` — CPA approves
- [ ] Journal Entry debits = credits for every entry
- [ ] User confirmation before posting any adjusting entry
- [ ] Portal updated after every step (incremental updates)

## Common Mistakes to Avoid

- Skipping accrual reversals from prior month → double-counts expenses
- Recording depreciation inconsistent with prior months without explanation
- Comparing a partial month to a full month in variance analysis
- Closing before all accounts are reconciled
- Posting adjusting entries directly instead of proposing for CPA review
- Forgetting to update the portal via `closeRun` after each step
