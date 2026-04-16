---
name: bank-reconciliation
description: Reconcile bank and credit card accounts — match transactions, identify discrepancies, resolve uncleared items, and validate account health scores. Use when the user mentions reconciliation, bank matching, uncleared items, or account health.
---

# Bank Reconciliation Skill

Match bank and credit card statement activity to QuickBooks records. Identify discrepancies, resolve uncleared items, and ensure account balances are accurate.

## Trigger

Activate when the user wants to:
- Reconcile a bank or credit card account
- Check account health scores
- Investigate uncleared or missing transactions
- Find duplicate transactions
- Clean up uncategorized transactions
- Verify bank balances match QB balances

## How Reconciliation Works

```
Bank Statement Balance
  - Outstanding checks (in QB, not yet cleared at bank)
  + Deposits in transit (in QB, not yet posted at bank)
  = QB Book Balance
```

If these don't match, there's a discrepancy to investigate.

## Workflow: Account Health Check

The fastest way to assess reconciliation status:

1. **Get accounts** — `qbMasterData(entityTypes=["account"])` → filter for Bank and Credit Card types
2. **Run health check** — `qbAccountHealth(accountId, startDate, endDate)` on each account
3. **Review flags**:
   - **Duplicates** — same amount + date + vendor (high severity if > $1,000)
   - **Uncategorized** — booked to "Ask My Accountant" or "Uncategorized" (high if > $500)
   - **Outliers** — amount exceeds 2 standard deviations from mean
   - **Past-due items** — transactions with dueDate older than 7 days
4. **Score** — 0-100 scale. Penalty: high flag = -5, medium = -3, low = -1

### Score Thresholds

| Score | Status | Action |
|-------|--------|--------|
| 95-100 | Audit-ready | No action needed |
| 90-94 | Good | Minor cleanup |
| 80-89 | Needs attention | Several issues to resolve |
| < 80 | Critical | Prioritize reconciliation immediately |

## Workflow: Full Account Reconciliation

For each Bank and Credit Card account:

### Step 1: Pull QB Transactions
```
qbFetchTransactions → all transactions for the account in the reconciliation period
```
Use report scan mode (omit transactionType, provide accountId + dates) to see all transaction types touching the account.

### Step 2: Pull Bank Data
```
bankFeed(action="fetch") → unprocessed bank transactions
```
These are the raw bank statement lines that need to be matched or recorded.

### Step 3: Match Transactions

For each bank statement line:
- **Exact match** — Same amount, same date (±2 days), same vendor → mark as reconciled
- **Partial match** — Amount matches but date/vendor differ → flag for review
- **No match** — Bank has it, QB doesn't → missing transaction, needs recording
- **QB-only** — QB has it, bank doesn't → outstanding check/deposit, or possible error

### Step 4: Resolve Discrepancies

**Missing in QB (bank has it, QB doesn't):**
1. Check `agentMemory` for vendor mapping
2. If known vendor → record using standard transaction workflow
3. If unknown vendor → `flagForReview` with aiReasoning: "Unmatched bank transaction — no QB record found"

**Missing in bank (QB has it, bank doesn't):**
1. If recent (< 5 days) → likely outstanding, will clear soon
2. If old (> 30 days) → investigate: was the check cashed? Did the transfer complete?
3. If very old (> 90 days) → may need voiding with CPA approval

**Duplicates detected:**
1. Identify which is the correct entry (check dates, amounts, memos)
2. `qbVoidTransaction` on the duplicate (requires CPA confirmation)
3. Note: voiding preserves audit trail, deleting does not

**Uncategorized transactions:**
1. Pull the specific transactions booked to "Ask My Accountant" or "Uncategorized"
2. Check `agentMemory` for vendor mappings
3. Re-categorize if confident, or `flagForReview` if uncertain

### Step 5: Verify Balance

After resolving all items:
1. Re-run `qbAccountHealth` to verify improved score
2. Compare QB balance to bank statement balance
3. Document any remaining reconciling items (outstanding checks, deposits in transit)

## Workflow: Investigate Specific Flags

When investigating a flagged item from health check:

1. **Duplicate flag** — `qbFetchTransactions` to pull both entries, compare details, confirm which to void
2. **Uncategorized flag** — Read transaction details, check `agentMemory`, re-categorize or flag
3. **Outlier flag** — Verify the amount is correct (not a data entry error), check if it's a valid large transaction
4. **Past-due flag** — Check if the item has cleared at the bank, investigate why it's still uncleared in QB

## Workflow: Credit Card Reconciliation

Credit cards follow the same process with one addition:

1. Run `qbAccountHealth` on the CC account
2. Match statement charges to QB expenses
3. **Statement balance check** — CC statement balance should match QB CC liability balance
4. Common issues:
   - Personal charges on business card → reclassify to Owner's Draw/Loan
   - Pending charges → will post with slight date differences
   - Returns/credits → verify they're recorded as negative amounts or vendor credits

## Safety Checklist

- [ ] All Bank and Credit Card accounts identified via `qbMasterData`
- [ ] Health check run on each account for the correct period
- [ ] Duplicate transactions confirmed before voiding (never void without verification)
- [ ] Uncategorized items resolved or flagged for CPA review
- [ ] Ending balance verified against bank statement
- [ ] All reconciling items documented

## Common Mistakes to Avoid

- Voiding a transaction without confirming it's actually a duplicate — check dates, amounts, and vendors carefully
- Ignoring old outstanding items — they may indicate a recording error, not just timing
- Reconciling against the wrong statement period
- Categorizing unfamiliar transactions without checking agent memory or flagging for review
- Forgetting credit card accounts — they need reconciliation too
- Not re-running health check after fixes to verify the score improved
