---
description: Budget vs actual analysis and variance reporting
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period]
---

Generate a budget vs actual analysis for: "$ARGUMENTS"

Default to current month if not specified.

**Note:** QuickBooks Online budgets exist in the UI but have NO API access.
Budgets cannot be read or written via the QBO API. This command uses
historical data to create a baseline "expected" budget and compares actual
performance against it. If the user has set up budgets in QBO and wants to
reference them, they'll need to provide the budget numbers manually or use
the `/reconcile` browser command to view them in the QBO UI.

Steps:

**Phase 1: Build the Baseline (Historical Average)**
1. Pull qbReports "ProfitAndLoss" for the last 3-6 months (use prior months)
2. Calculate average monthly values for each account line
3. This becomes the "expected" baseline for comparison

**Phase 2: Pull Actual Data**
4. Pull qbReports "ProfitAndLoss" for the target period (current month or specified)

**Phase 3: Variance Analysis**
5. For each P&L line item, calculate:
   - Historical average (baseline/budget)
   - Actual this period
   - $ variance (Actual - Budget)
   - % variance
6. Flag significant variances:
   - Revenue shortfall >10% → WARNING
   - Expense overrun >25% AND >$1,000 → WARNING
   - Any new expense category not in baseline → REVIEW
   - Any baseline category that dropped to $0 → NOTE (missing transaction?)

**Phase 4: Presentation**

1. **Headline**: "Revenue is [above/below] trend by $X; expenses are [above/below] by $Y"
2. **Summary Table**:

| Category | 3-Mo Avg | Actual | $ Var | % Var | Status |
|----------|----------|--------|-------|-------|--------|
| Total Revenue | $X | $Y | +/- | % | OK/WARN |
| Total COGS | $X | $Y | +/- | % | OK/WARN |
| Gross Profit | $X | $Y | +/- | % | OK/WARN |
| [Each major expense] | ... | ... | ... | ... | ... |
| Net Income | $X | $Y | +/- | % | OK/WARN |

3. **Material Variances**: For each significant variance, drill into
   qbReports "TransactionList" to explain what drove it. Cite specific
   transactions — "Advertising up $3,200 due to new Google Ads campaign
   started March 5 ($2,800) and Facebook boost ($400)"
4. **Trend Direction**: Is each major category trending up, stable, or down
   over the 3-6 month window?
5. **Recommendations**: Actionable items based on variances

**If user has provided an actual budget** (mentions specific targets):
Use their numbers instead of historical averages. Compare against their
stated targets and flag where actuals diverge.
