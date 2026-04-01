# /budget

Compare actual financial performance against budgets in QuickBooks.

## Usage
```
/budget                        # Current month budget vs actuals
/budget ytd                    # Year-to-date budget vs actuals
/budget q1 2026                # Quarterly comparison
/budget create                 # Create or update a budget in QB
```

## Behavior

Use the **CFO** agent with the **Financial Analysis** skill.

### View Mode (default)
1. Fetch budget data: `qbBudget(action="read")`
2. Pull actual P&L for the same period: `qbReports(reportType="ProfitAndLoss")`
3. Calculate variances for each line item:
   - **Dollar variance** = Actual - Budget
   - **Percentage variance** = (Actual - Budget) / Budget
4. Present a comparison table:

| Category | Budget | Actual | $ Variance | % Variance | Status |
|----------|--------|--------|------------|------------|--------|

5. Flag significant variances:
   - Revenue under budget by > 10% → warning
   - Any expense over budget by > 15% → alert
   - Net income below budget → highlight root causes
6. Provide 2-3 actionable insights on the largest variances

### Create Mode
When called with `create`:
1. Ask user for: fiscal year, budget period (Monthly/Quarterly/Annual)
2. Optionally pull prior year actuals as a starting baseline: `qbReports(reportType="ProfitAndLoss")` for the prior year
3. Present the baseline for user adjustment
4. Create budget: `qbBudget(action="create")` with the finalized numbers

## Safety
- Budget creation requires explicit user confirmation
- Never overwrite an existing budget without asking — create a new one or update specific line items
- Always show the full comparison before making recommendations
