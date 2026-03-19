# Financial Warning Signs — Autonomous Detection and Response

## Core Principle: Always Check, Always Surface

Warning signs should be checked on EVERY financial analysis, regardless of
what the user specifically asked about. If you detect a warning sign while
answering a P&L question, surface it. Don't wait to be asked.

Frame unsolicited warnings as: "While reviewing [X], I also noticed..."

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
- <3 months: CRITICAL — surface immediately with urgency
- 3-6 months: WARNING — surface with recommended actions
- 6-12 months: MONITOR — mention in health checks
- >12 months: OK — note positively

**Response:** "Cash runway is approximately [X] months at current burn rate
of [$Y/month]. [Action based on threshold.]"

### Declining Cash Balance

**Detection:** Cash balance decreased >15% month-over-month for 2+
consecutive months.

**Auto-investigate:** Pull transaction-level data to identify the cause
(increased spending? delayed collections? owner draws?).

**Response:** "Cash balance has declined for [N] consecutive months,
from [$X] to [$Y]. [Root cause from investigation]. [Recommendation.]"

## Profitability Warnings

### Gross Margin Compression

**Detection:** Gross margin % dropped >3 percentage points from prior period.

**Calculation:**
```
Gross Margin % = (Revenue - COGS) / Revenue × 100
Change = Current GM% - Prior GM%
```

**Auto-investigate:** Drill into COGS transactions to identify whether
input costs rose, pricing changed, or product/service mix shifted.

**Response:** "Gross margin declined from [X]% to [Y]%, a [Z]pp compression.
[Root cause from investigation]. [Recommendation.]"

### Negative Net Income

**Detection:** Net income is negative (net loss).

**Auto-investigate:** Determine if this is a first occurrence or recurring,
and identify the primary drivers.

**Response:** "[If first time: 'The business reported its first net loss of
[$X] — a change from [$Y] profit last period. Primary drivers: [causes].'
If recurring: 'This is the [Nth] consecutive month of losses, totaling [$Y].
The trend is [worsening/improving]. Primary driver: [cause].']"

### Operating Expense Growth Exceeding Revenue

**Detection:** OpEx growth rate > Revenue growth rate for 2+ periods.

**Response:** "Operating expenses are growing faster than revenue —
expenses up [X]% vs revenue up [Y]%. If this trend continues, margins
will continue to compress. [Specific categories driving the gap from
drill-down.]"

## Receivables Warnings

### Aging AR

**Detection:** Receivables >60 days represent >20% of total AR.

**Auto-investigate:** Identify the specific customers and invoices driving
the aging.

**Response:** "[$X] ([Y]%) of accounts receivable is over 60 days past due.
Top overdue: [list with specific invoice details]. Recommended actions:
[prioritized by amount]."

### Growing AR Relative to Revenue

**Detection:** AR / Monthly Revenue ratio increasing over 3+ months.

**Calculation:**
```
Days Sales Outstanding (DSO) = (AR / Revenue) × Days in Period
```

**Response:** "Days Sales Outstanding has increased from [X] to [Y] days
over the past [N] months. Customers are taking longer to pay, tying up
[$X] in working capital. [Specific customers driving the trend.]"

### Single Customer Concentration

**Detection:** Any single customer >30% of total revenue (from CustomerIncome).

**Response:** "[$Customer] represents [X]% of total revenue ([$Y] of [$Z]).
This creates significant concentration risk. [Compare to prior period —
is concentration increasing or decreasing?]"

## Payables Warnings

### Overdue Payables

**Detection:** AP >60 days represents >30% of total AP.

**Auto-investigate:** Identify which vendors and bills are overdue.

**Response:** "[$X] in payables is over 60 days past due. Top overdue:
[list with specific bill details]. This may damage vendor relationships
or result in late fees. [Recommendation based on cash position.]"

### Rising AP

**Detection:** AP increased >25% month-over-month without corresponding
revenue increase.

**Response:** "Accounts payable increased [X]% to [$Y] without a
corresponding revenue increase. [Root cause from investigation:
stretching payments, new vendor commitments, etc.]"

## Expense Warnings

### Category Spike

**Detection:** Any expense category increased >25% from prior period
AND the increase is >$2,000.

**Auto-investigate:** Pull transactions in that category to identify
the specific items causing the spike.

**Response:** "[Category] expenses jumped [X]% ([$Y]). [Always cite
the specific transactions: 'Driven by [transaction 1] ([$X]) and
[transaction 2] ([$Y]).']"

### New Large Expense

**Detection:** A new expense account appeared that wasn't used in the
prior period, with >$5,000 in activity.

**Auto-investigate:** Pull all transactions in the new account.

**Response:** "New expense category '[Account]' appeared this period
with [$X] in charges across [N] transactions. [List the transactions.]
Verify this is correctly categorized."

## When to Surface Warnings

- **ALWAYS** check during any financial analysis, even if user didn't ask
- **ALWAYS** surface detected warnings — don't wait to be asked
- **ALWAYS** auto-investigate to provide root cause, not just the alert
- Frame unsolicited warnings helpfully: "While reviewing [X], I also noticed..."
- Do NOT repeatedly warn about the same issue within a single conversation
- Prioritize warnings by severity: CRITICAL > WARNING > MONITOR
