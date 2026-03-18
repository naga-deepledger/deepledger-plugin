# Variance Analysis — Autonomous Methodology and Templates

## Core Principle: Complete Analysis, No Intermediate Questions

Pull all data, calculate all variances, drill into material items, and
present complete findings. Never ask "should I investigate further?" —
always investigate by default.

## Period Comparison Framework

### Automatic Period Detection

| User Request | Primary Comparison | Secondary Comparison |
|-------------|-------------------|---------------------|
| "This month" | Current month vs prior month | vs same month last year |
| "This quarter" | Current quarter vs prior quarter | vs same quarter last year |
| "This year" | YTD vs prior YTD | Full prior year |
| "How am I doing" | Current month vs prior month | + YTD vs prior YTD |
| No period specified | Current month vs prior month | (default) |

Determine the right comparison autonomously — don't ask the user which
periods to compare.

### Materiality Thresholds

Not every variance deserves commentary. Focus on variances that are:

**Material by dollar amount:**
- Revenue line items: >$5,000 change
- Expense categories: >$2,000 change
- Net income: any significant change

**Material by percentage:**
- >10% change on large accounts (>$10K)
- >25% change on medium accounts ($1K-$10K)
- >50% change on small accounts (<$1K)

**Always material regardless of size:**
- New accounts that didn't exist in prior period
- Accounts that dropped to zero
- Sign changes (positive to negative or vice versa)

### Variance Calculation

```
$ Change = Current Period - Prior Period
% Change = (Current - Prior) / |Prior| × 100

If Prior = 0 and Current > 0: "New this period"
If Prior > 0 and Current = 0: "None this period (was $X)"
```

## Automatic Drill-Down

When a variance is material, automatically investigate:

1. Identify the account(s) driving the variance
2. Use qbReports (TransactionList) filtered by that account and date range
3. Look for:
   - Large individual transactions (>50% of the variance)
   - New recurring charges
   - Missing recurring revenue
   - One-time items that distort the trend
4. Include specific transaction references in the narrative

Do NOT ask "would you like me to drill into this?" — just do it.

## Narrative Templates

### Revenue Increase
"Revenue grew [$ amount] ([%]) to [$total] this [period]. The increase was
primarily driven by [top contributor] ([$ amount]). [Additional context about
customer mix, new clients, or seasonal factors if visible from data]."

### Revenue Decrease
"Revenue declined [$ amount] ([%]) to [$total] this [period]. The main driver
was [top contributor] ([$ amount]). [Specific transactions or customers
identified from drill-down]. [Recommendation based on findings.]"

### Expense Increase
"[Category] expenses increased [$ amount] ([%]) to [$total]. [Cite specific
transactions from drill-down: 'This was driven by [transaction detail].'
Always include the specific cause, never just state the number.]"

### Margin Compression
"Gross margin declined from [prior]% to [current]% — a [change]pp compression.
This means [$ amount] less profit per dollar of revenue. [Root cause from
drill-down: COGS increase, pricing change, or mix shift.]"

### Net Income Impact
"Net income [increased/decreased] by [$ amount] ([%]). Key drivers:
1. [Largest contributing factor] ([±$ amount])
2. [Second factor] ([±$ amount])
[Overall: business is [improving/stable/declining] relative to prior period.]"

## Multi-Period Trends

When the user asks "how am I doing" or wants a health check:

1. Pull 3-6 months of P&L data
2. Calculate month-over-month trends for:
   - Total revenue
   - Gross margin %
   - Total operating expenses
   - Net income
3. Identify direction: growing, stable, or declining
4. Calculate growth rates: monthly and annualized
5. Present as a trend narrative, not just numbers
6. Surface any inflection points (e.g., "Revenue peaked in [month] and has
   declined 3 consecutive months since")
