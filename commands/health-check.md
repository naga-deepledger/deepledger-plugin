# /health-check

Run a quick financial health check across all bank and credit card accounts.

## Usage
```
/health-check                  # Full health check on all accounts
/health-check checking         # Check a specific account
/health-check --detailed       # Include P&L, AR/AP, and cash runway
```

## Behavior

Use the **CFO** agent with the **Financial Analysis** skill.

### Quick Mode (default)
1. Get all Bank and Credit Card accounts: `qbMasterData(entityType="Account")`
2. Assess each account using existing tools:
   - `qbFetchTransactions` (accountId + date range) — scan for duplicates (same vendor + amount + date within ±3 days) and entries booked to "Uncategorized" or "Ask My Accountant"
   - `qbReports(reportType="GeneralLedger")` scoped to the account — review the period's activity for outliers (amounts far outside the account's normal range)
3. Compute a health score per account (start at 100; deduct 5 per high-severity flag, 3 per medium, 1 per low) and present a scorecard:

| Account | Health Score | Uncleared | Duplicates | Uncategorized | Outliers |
|---------|-------------|-----------|------------|---------------|----------|

4. Flag any account with score < 90 as needing attention
5. Target: all accounts >= 90 (>= 95 for audit-ready)

### Detailed Mode (--detailed)
In addition to the quick mode, also:
1. `qbReports(reportType="ProfitAndLoss")` — current month headline
2. `qbReports(reportType="AgedReceivables")` — overdue AR total
3. `qbReports(reportType="AgedPayables")` — overdue AP total
4. Calculate cash runway from Cash Flow data
5. Present a full dashboard:

```
Health Check — [Client Name] — [Date]
═══════════════════════════════════════
Accounts:     [X/Y] healthy (score >= 90)
Revenue MTD:  $XX,XXX (up/down XX% vs prior month)
Net Income:   $XX,XXX (margin: XX%)
Cash on Hand: $XX,XXX (runway: X months)
AR Overdue:   $XX,XXX (X invoices)
AP Due Soon:  $XX,XXX (X bills)
Pending Work: X tasks, Y reviews
═══════════════════════════════════════
```

6. Top 3 issues to address

### Specific Account Mode
When an account name is given:
1. Find the account via `qbMasterData`
2. Run the same `qbFetchTransactions` + `qbReports(reportType="GeneralLedger")` checks on just that account
3. Show detailed breakdown: uncleared items, duplicates, uncategorized, outliers
4. List specific flagged transactions for investigation

## Thresholds
- **Green** (>= 95): Audit-ready, no action needed
- **Yellow** (90-94): Good shape, minor items to clean up
- **Orange** (80-89): Needs attention, several issues
- **Red** (< 80): Critical, prioritize reconciliation immediately
