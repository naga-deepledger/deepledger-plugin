# /close-books

Run the month-end close workflow — reconciliation, accruals, depreciation, and final review.

## Usage
```
/close-books                   # Close the most recent completed month
/close-books march 2026        # Close a specific month
/close-books status            # Check current close status without making changes
```

## Behavior

Use the **Accountant** agent with the **Bookkeeping** skill.
Follow the `getGuide(guideType="month_end_closing")` workflow.

### Step 1: Reconcile All Accounts
- `qbMasterData(entityType="Account")` → get all Bank and Credit Card accounts
- `qbReconciliationCheck` on each account for the closing month
- Report health scores. Target: >= 90 on all accounts
- If any account < 90, investigate and resolve issues before continuing

### Step 2: Review Outstanding AP & AR
- `qbReports(reportType="AgedPayables")` — verify all bills are in the correct period
- `qbReports(reportType="AgedReceivables")` — verify all invoices are in the correct period
- Flag past-due items. AR over 90 days may need bad debt write-off

### Step 3: Record Accruals & Deferrals
- Check prior month JEs for recurring accruals that need reversal
- Record new accruals via `qbJournalEntry`:
  - Accrued expenses: Debit Expense, Credit Accrued Liabilities
  - Prepaid amortization: Debit Expense, Credit Prepaid Asset
  - Deferred revenue recognition as appropriate
- Use `qbBatch` for multiple JEs

### Step 4: Record Depreciation
- Review prior depreciation JEs for amounts (should be consistent monthly)
- Record: Debit Depreciation Expense, Credit Accumulated Depreciation
- Include asset description in memo

### Step 5: Review Financial Statements
- `qbReports(reportType="ProfitAndLoss")` — compare to prior month and same month last year
- `qbReports(reportType="BalanceSheet")` — verify it balances
- `qbReports(reportType="CashFlow")` — reconcile to cash change on Balance Sheet
- Flag variances > 5% in gross margin vs prior periods

### Step 6: Final Trial Balance
- `qbReports(reportType="TrialBalance")` — confirm debits = credits
- If out of balance, investigate one-sided entries or posting errors

### Report Results
Present a close summary:
- All account health scores
- Adjusting entries made
- Final P&L headline numbers
- Any issues requiring CPA attention

## Safety
- All changes require user confirmation before posting
- Never close books before all accounts are reconciled
- Document every adjusting entry with clear memo
- Reversing prior-month accruals is critical — don't skip it
