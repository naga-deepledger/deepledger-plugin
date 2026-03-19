# Recurring Transaction Detection & Management

## Purpose

Detect recurring patterns in transaction history to:
1. Auto-suggest when a recurring transaction is due
2. Flag when an expected recurring transaction is missing
3. Reduce data entry by pre-filling recurring transaction details
4. Catch duplicate recordings of recurring items

## Detection Logic

### How to Identify Recurring Transactions

When fetching vendor/customer history via qbFetchTransactions, look for:

**Strong recurring signal (high confidence):**
- 3+ transactions to same vendor
- Similar amounts (within ±10%)
- Regular interval (monthly ±5 days, weekly ±2 days, quarterly ±10 days)
- Same expense account each time

**Moderate recurring signal:**
- 2 transactions to same vendor
- Similar amounts (within ±15%)
- Approximately one interval apart (monthly, weekly, etc.)
- Same expense account

**Not recurring:**
- Varying amounts with no pattern
- Irregular timing
- Different accounts each time

### Common Recurring Transaction Types

| Category | Typical Frequency | Examples |
|----------|------------------|---------|
| Rent/Lease | Monthly (1st or 15th) | Office rent, equipment lease |
| Software/SaaS | Monthly or Annual | Slack, Zoom, AWS, GitHub, Adobe |
| Insurance | Monthly or Quarterly | Business liability, health, property |
| Utilities | Monthly | Electric, gas, water, internet, phone |
| Payroll | Bi-weekly or Semi-monthly | Wages, payroll taxes |
| Loan Payments | Monthly | Business loans, lines of credit |
| Subscriptions | Monthly or Annual | Trade publications, memberships |
| Cleaning/Maintenance | Weekly or Monthly | Janitorial, landscaping |
| Marketing | Monthly | Ad spend retainers, SEO services |
| Professional Services | Monthly or Quarterly | Bookkeeping, legal retainer |

## Proactive Detection During Recording

When the user records a transaction, automatically check if it matches a
recurring pattern:

### During `/record`:
1. After looking up vendor history (Step 3 of normal workflow), scan for
   recurring pattern
2. If recurring pattern detected:
   - Auto-fill amount, account, and description from most recent occurrence
   - Note in proposal: "(recurring — matches monthly pattern of $X to [account])"
   - If amount differs >10% from pattern, note: "(amount differs from usual $X)"
3. If the same transaction was already recorded this period:
   - Flag as potential duplicate with stronger confidence than normal
   - "This vendor already has a $X transaction on [date] this month —
     is this an additional charge or a duplicate?"

### During `/close-books`:
1. Build a list of expected recurring transactions based on the last 3 months
2. Check each one against the closing month's transactions
3. Flag any that are MISSING:
   - "Expected monthly $X payment to [vendor] for [category] — no matching
     transaction found in [month]. Was this paid? Or should it be accrued?"
4. Flag any that appear DOUBLED:
   - "Two payments to [vendor] this month ($X on [date1] and $Y on [date2])
     — usually only one per month. Verify both are correct."

## Recurring Transaction Summary Table

When useful (e.g., during health checks or close), present:

| Vendor | Amount | Frequency | Account | Last Recorded | Next Expected |
|--------|--------|-----------|---------|---------------|---------------|
| Landlord | $3,500 | Monthly | Rent | Mar 1 | Apr 1 |
| AWS | $450 | Monthly | Software | Mar 3 | Apr 3 |
| Gusto | $12,000 | Bi-weekly | Payroll | Mar 15 | Mar 29 |

## Integration Points

This reference is used by:
- **accountant agent**: Pre-fill recurring transaction details during recording
- **cfo agent**: Flag missing recurring items during analysis
- **/record command**: Auto-detect pattern and pre-fill
- **/close-books command**: Validate all expected recurring items are recorded
- **/health-check command**: Surface missing or unusual recurring patterns
