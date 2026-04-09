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

### Structured Output — closeRun MCP Tool

All results MUST be written via the `closeRun` MCP tool (not memory, not execute_sql).

#### Before starting:
```
closeRun(operation="start", period="2026-03", periodLabel="March 2026")
```

#### After each step — update incrementally so the portal shows live progress:
```
closeRun(operation="updateStep", step={id: "reconciliation", label: "Reconcile All Accounts", status: "passed", ...})
closeRun(operation="updateChecks", checks=[{id: 7, label: "Bank Reconciliation", status: "pass", value: "95/100", ...}])
closeRun(operation="addEntries", adjustingEntries=[{type: "depreciation", description: "...", ...}])
closeRun(operation="setFinancials", financials={revenue: 85000, expenses: 62000, ...})
```

#### When complete:
```
closeRun(operation="complete", score=14, actionItems=2)
```

The `result` column uses this schema:

```json
{
  "steps": [
    {
      "id": "reconciliation",
      "label": "Reconcile All Accounts",
      "status": "passed|warning|failed|running|pending|skipped",
      "started_at": "ISO timestamp",
      "completed_at": "ISO timestamp",
      "details": "Human-readable summary of what was found"
    },
    { "id": "ap_ar", "label": "Review AP & AR", ... },
    { "id": "accruals", "label": "Record Accruals & Deferrals", ... },
    { "id": "depreciation", "label": "Record Depreciation", ... },
    { "id": "statements", "label": "Review Financial Statements", ... },
    { "id": "trial_balance", "label": "Final Trial Balance", ... }
  ],
  "checks": [
    {
      "id": 1,
      "label": "Trial Balance",
      "status": "pass|warning|action_needed|unknown",
      "value": "$0.00 difference",
      "detail": "Total debits $125,432.00 = Total credits $125,432.00",
      "fix_action": null
    },
    ...all 16 checks...
  ],
  "adjusting_entries": [
    {
      "type": "accrual|depreciation|reversal|reclassification",
      "description": "Monthly depreciation - Office Equipment",
      "debit_account": "Depreciation Expense",
      "credit_account": "Accumulated Depreciation",
      "amount": 450.00,
      "qb_txn_id": null,
      "status": "proposed|approved|posted|rejected"
    }
  ],
  "financials": {
    "revenue": 85000.00,
    "expenses": 62000.00,
    "net_income": 23000.00,
    "total_assets": 340000.00,
    "total_liabilities": 120000.00,
    "equity": 220000.00,
    "cash": 45000.00,
    "prior_month": {
      "revenue": 80000.00,
      "expenses": 58000.00,
      "net_income": 22000.00
    }
  },
  "score": 14,
  "total": 16,
  "action_items": 2
}
```

Note: The `complete` operation sets status to `needs_review` — only a CPA can close in the portal.

### Step 1: Reconcile All Accounts
- `qbMasterData(entityType="Account")` → get all Bank and Credit Card accounts
- `qbAccountHealth` on each account for the closing month
- Report health scores. Target: >= 90 on all accounts
- If any account < 90, investigate and resolve issues before continuing
- **Update close_runs**: Set step `reconciliation` status + update checks 7 (Bank Reconciliation)

### Step 2: Review Outstanding AP & AR
- `qbReports(reportType="AgedPayables")` — verify all bills are in the correct period
- `qbReports(reportType="AgedReceivables")` — verify all invoices are in the correct period
- Flag past-due items. AR over 90 days may need bad debt write-off
- **Update close_runs**: Set step `ap_ar` status + update checks 5 (AP), 6 (AR)

### Step 3: Record Accruals & Deferrals
- Check prior month JEs for recurring accruals that need reversal
- Record new accruals via `qbJournalEntry`:
  - Accrued expenses: Debit Expense, Credit Accrued Liabilities
  - Prepaid amortization: Debit Expense, Credit Prepaid Asset
  - Deferred revenue recognition as appropriate
- Use `qbBatch` for multiple JEs
- **Update close_runs**: Set step `accruals` status + add to `adjusting_entries` array + update checks 11 (Prepaid), 13 (Accruals)
- Proposed entries should have `status: "proposed"` — wait for CPA approval in portal before posting

### Step 4: Record Depreciation
- Review prior depreciation JEs for amounts (should be consistent monthly)
- Record: Debit Depreciation Expense, Credit Accumulated Depreciation
- Include asset description in memo
- **Update close_runs**: Set step `depreciation` status + add to `adjusting_entries` + update check 12

### Step 5: Review Financial Statements
- `qbReports(reportType="ProfitAndLoss")` — compare to prior month and same month last year
- `qbReports(reportType="BalanceSheet")` — verify it balances
- `qbReports(reportType="CashFlow")` — reconcile to cash change on Balance Sheet
- Flag variances > 5% in gross margin vs prior periods
- **Update close_runs**: Set step `statements` status + populate `financials` object + update checks 9 (Large/Unusual), 10 (Revenue Recognition)

### Step 6: Final Trial Balance
- `qbReports(reportType="TrialBalance")` — confirm debits = credits
- If out of balance, investigate one-sided entries or posting errors
- **Update close_runs**: Set step `trial_balance` status + update check 1 (Trial Balance)
- Also update remaining checks: 2 (Undeposited Funds), 3 (Uncategorized), 4 (Suspense), 8 (Recurring), 14 (Payroll), 15 (Sales Tax), 16 (Changes Since Last Close)

### Final: Compute Score & Set Status
- Count checks with status "pass" → that's the `score` out of 16
- Count checks with status "action_needed" → that's `action_items`
- Update close_runs with final `score`, `total`, `action_items`
- Set `status` to `needs_review`

### Report Results
Present a close summary:
- All account health scores
- Adjusting entries made (or proposed)
- Final P&L headline numbers
- Any issues requiring CPA attention

### 16 Checks Reference

| # | Label | What to Check |
|---|-------|---------------|
| 1 | Trial Balance | Debits = credits, no out-of-balance |
| 2 | Undeposited Funds | Balance $0 or near-zero |
| 3 | Uncategorized Transactions | Nothing in Uncategorized/Ask My Accountant |
| 4 | Suspense Accounts | Clearing accounts at zero |
| 5 | AP Accuracy | Open bills match obligations, none >90 days |
| 6 | AR Accuracy | Open invoices match expectations, none >90 days |
| 7 | Bank Reconciliation | All bank/CC accounts reconciled |
| 8 | Recurring Transactions | All expected recurring items posted |
| 9 | Large/Unusual Entries | No unexplained amounts >2x average |
| 10 | Revenue Recognition | Revenue in correct period |
| 11 | Prepaid Expenses | Amortization entries recorded |
| 12 | Depreciation | Fixed asset depreciation posted |
| 13 | Accruals | Known incurred-but-not-billed expenses accrued |
| 14 | Payroll | All entries recorded and categorized |
| 15 | Sales Tax | Collected tax accounted for |
| 16 | Changes Since Last Close | No unapproved edits to closed transactions |

## Safety
- All changes require user confirmation before posting
- Never close books before all accounts are reconciled
- Document every adjusting entry with clear memo
- Reversing prior-month accruals is critical — don't skip it
- Adjusting entries default to `proposed` status — CPA approves in portal
