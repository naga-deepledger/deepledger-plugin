# Financial Warning Signs — Detection and Response

## Cash Position Warnings

### Low Cash Runway

**Detection:** Monthly burn rate (negative operating cash flow) exceeds
cash reserves when projected forward.

**Calculation:**
```
Monthly burn = Average of last 3 months' operating cash flow (if negative)
Current cash = Total bank account balances from Balance Sheet
Runway = Current cash / |Monthly burn|
```

**Thresholds:**
- <3 months: CRITICAL — immediate action needed
- 3-6 months: WARNING — plan for funding or cost reduction
- 6-12 months: MONITOR — healthy but watch trend
- >12 months: OK

**Response:** "Current cash runway is approximately [X] months based on
average monthly burn of [$Y]. [Recommendation based on threshold.]"

### Declining Cash Balance

**Detection:** Cash balance decreased >15% month-over-month for 2+
consecutive months.

**Response:** "Cash balance has declined for [N] consecutive months,
from [$X] to [$Y]. This trend suggests [spending exceeds revenue /
large one-time expenses / AR collection issues]. Recommend reviewing
cash flow statement for details."

## Profitability Warnings

### Gross Margin Compression

**Detection:** Gross margin % dropped >3 percentage points from prior period.

**Calculation:**
```
Gross Margin % = (Revenue - COGS) / Revenue × 100
Change = Current GM% - Prior GM%
```

**Response:** "Gross margin declined from [X]% to [Y]%, a [Z]pp compression.
This means [$amount] less profit per dollar of revenue. Investigate: are
input costs rising? Has pricing changed? Is the product/service mix shifting?"

### Negative Net Income

**Detection:** Net income is negative (net loss).

**Response:** "The business reported a net loss of [$X] this period.
[If first time: 'This is a change from profitable last period — investigate
what changed.' If ongoing: 'This is the [Nth] consecutive month of losses.
Current accumulated loss: [$Y].']"

### Operating Expense Growth Exceeding Revenue

**Detection:** OpEx growth rate > Revenue growth rate for 2+ periods.

**Response:** "Operating expenses are growing faster than revenue —
expenses up [X]% vs revenue up [Y]%. If this trend continues, margins
will continue to compress. Review expense categories for optimization."

## Receivables Warnings

### Aging AR

**Detection:** Receivables >60 days represent >20% of total AR.

**Response:** "[$X] ([Y]%) of accounts receivable is over 60 days past due.
Top overdue accounts: [list]. Recommend: follow up on collections, consider
offering early payment discounts, review credit terms."

### Growing AR Relative to Revenue

**Detection:** AR / Monthly Revenue ratio increasing over 3+ months.

**Calculation:**
```
Days Sales Outstanding (DSO) = (AR / Revenue) × Days in Period
```

**Response:** "Days Sales Outstanding has increased from [X] to [Y] days
over the past [N] months, meaning customers are taking longer to pay.
This ties up working capital. Current AR: [$X]."

### Single Customer Concentration

**Detection:** Any single customer >30% of total revenue (from CustomerIncome report).

**Response:** "[$Customer] represents [X]% of total revenue ([$Y] of [$Z]).
This creates concentration risk — if this customer reduces spending or
leaves, it would significantly impact the business. Consider diversifying
the customer base."

## Payables Warnings

### Overdue Payables

**Detection:** AP >60 days represents >30% of total AP.

**Response:** "[$X] in payables is over 60 days past due. This may damage
vendor relationships, result in late fees, or indicate cash flow problems.
Top overdue: [list]. Prioritize payment or negotiate extended terms."

### Rising AP

**Detection:** AP increased >25% month-over-month without corresponding
revenue increase.

**Response:** "Accounts payable increased [X]% this month to [$Y] without
a corresponding revenue increase. This suggests the business is stretching
payments — monitor cash position."

## Expense Warnings

### Category Spike

**Detection:** Any expense category increased >25% from prior period
AND the increase is >$2,000.

**Response:** "[Category] expenses jumped [X]% ([$Y]) this period.
[If specific large transaction found: 'Driven by: [description].'
If not: 'Review transactions in this category for unusual items.']"

### New Large Expense

**Detection:** A new expense account appeared that wasn't used in the
prior period, with >$5,000 in activity.

**Response:** "New expense category '[Account]' appeared this period
with [$X] in charges. This wasn't present last period. Verify this is
correctly categorized and expected."

## When to Surface Warnings

- **Always** during financial health checks ("how is my business doing")
- **Always** in P&L analysis when the warning condition is detected
- **Proactively** mention cash runway in any cash-related analysis
- **Proactively** mention aging in any AR/AP report
- Do NOT repeatedly warn about the same issue within a single conversation
