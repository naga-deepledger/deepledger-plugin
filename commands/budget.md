---
description: Budget vs actual analysis and variance reporting
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period]
---

Generate a budget vs actual analysis for: "$ARGUMENTS"

Default to current month if not specified.

**API Status:** QuickBooks budgets are **read-only via API** — you can query
and read existing budgets but cannot create or edit them (that's UI-only).
This means:
- If the user has set up a budget in QBO → try to read it via API first
- If no QBO budget exists → fall back to historical baseline approach
- To create/edit budgets → direct user to QBO UI (Settings → Budgets)

Steps:

**Phase 1: Try to Read QBO Budget (if available)**
1. Attempt to query for existing budgets via qbFetchTransactions or qbReports
   (look for Budget entity). If a qbBudget MCP tool exists, use that.
2. If a budget is found → use those numbers as the "Budget" column
3. If no budget exists or API doesn't support reading it → proceed to Phase 2

**Phase 2: Build Historical Baseline (fallback)**
1. Pull qbReports "ProfitAndLoss" for the last 3-6 months (prior months)
2. Calculate average monthly values for each account line
3. This becomes the "expected" baseline for comparison
4. Label the column "3-Mo Avg" (not "Budget") to be transparent

**Phase 3: Pull Actual Data**
5. Pull qbReports "ProfitAndLoss" for the target period (current month or specified)

**Phase 4: Variance Analysis**
6. For each P&L line item, calculate:
   - Budget or historical average
   - Actual this period
   - $ variance (Actual - Budget)
   - % variance
7. Flag significant variances:
   - Revenue shortfall >10% → WARNING
   - Expense overrun >25% AND >$1,000 → WARNING
   - Any new expense category not in baseline → REVIEW
   - Any baseline category that dropped to $0 → NOTE (missing transaction?)

**Phase 5: Presentation**

1. **Headline**: "Revenue is [above/below] [budget/trend] by $X; expenses are [above/below] by $Y"
2. **Source note**: Clearly state whether comparing to QBO budget or historical baseline
3. **Summary Table**:

| Category | Budget/3-Mo Avg | Actual | $ Var | % Var | Status |
|----------|----------------|--------|-------|-------|--------|
| Total Revenue | $X | $Y | +/- | % | OK/WARN |
| Total COGS | $X | $Y | +/- | % | OK/WARN |
| Gross Profit | $X | $Y | +/- | % | OK/WARN |
| [Each major expense] | ... | ... | ... | ... | ... |
| Net Income | $X | $Y | +/- | % | OK/WARN |

4. **Material Variances**: For each significant variance, drill into
   qbReports "TransactionList" to explain what drove it. Cite specific
   transactions — "Advertising up $3,200 due to new Google Ads campaign
   started March 5 ($2,800) and Facebook boost ($400)"
5. **Trend Direction**: Is each major category trending up, stable, or down
   over the 3-6 month window?
6. **Recommendations**: Actionable items based on variances

**If user has provided an actual budget** (mentions specific targets):
Use their numbers instead of historical averages or QBO budget. Compare
against their stated targets and flag where actuals diverge.
