# Duplicate Detection — Autonomous Methodology

## Core Principle: Silent When Clear, Ask When Ambiguous

Duplicate checking should be invisible to the user in the common case
(no duplicates found). Only surface results when a potential match is
detected and the user needs to make a decision.

## Why Check for Duplicates

Double-recording transactions is one of the most common bookkeeping errors.
It inflates expenses, revenue, or both, leading to incorrect financial
statements. Always check before creating.

## When to Check

Before creating ANY of these transaction types:
- qbBill
- qbExpense
- qbInvoice
- qbSalesReceipt
- qbJournalEntry
- qbDeposit
- qbRefundReceipt
- qbCredit

Do NOT need to check for:
- qbBillPayment (linked to specific bill)
- qbReceivePayment (linked to specific invoice)
- qbVoidTransaction (modifying existing)

## Autonomous Duplicate Handling

### Step 1: Query Existing Transactions (silent)

Use qbFetchTransactions with these filters:
- **Vendor/Customer**: Same entity name
- **Date range**: ±3 days from the intended transaction date
- **Transaction type**: Same type if possible

### Step 2: Evaluate Results (autonomous decision)

**No matches found:**
→ Proceed silently. Do NOT tell the user "no duplicates found" — this
  adds noise without value. Just move on to the proposal.

**Exact match (same entity, same date, same amount within ±1%):**
→ HIGH confidence duplicate. Show it to the user and ask:
  "This appears to be a duplicate of [existing transaction]. Is this
  a separate, new transaction?"

**Near match (same entity, ±3 days, amount within ±5%):**
→ MEDIUM confidence. Show it and ask:
  "A similar transaction exists. Is this new or the same?"
  Display the existing transaction details for comparison.

**Weak match (same entity only, different date/amount):**
→ LOW confidence — NOT a duplicate. Proceed silently.
  Recent transactions from the same vendor are expected and normal.

### Step 3: Act on Decision

- User confirms it's new → proceed with creation
- User says it's a duplicate → abort, reference the existing transaction
- User doesn't respond clearly → ask once more, then wait

## Fuzzy Name Matching

Vendor/customer names may not match exactly:
- "Office Depot" vs "Office Depot Inc"
- "Amazon" vs "Amazon.com" vs "AMZN"
- "John's Plumbing" vs "Johns Plumbing LLC"

When searching, use partial name matches. If the vendor/customer lookup
returns multiple similar results, select the closest match if it's
obvious (e.g., "Office Depot" matches "Office Depot Inc"). Only ask
when genuinely ambiguous (e.g., "Johnson" matches "Johnson LLC" and
"Johnson & Associates" — two different entities).

## Amount Tolerance

Allow ±5% tolerance because:
- Tax may or may not be included
- Shipping charges may vary
- Currency rounding differences
- Partial payments

## Common Duplicate Scenarios

1. **Bank feed + manual entry**: User records an expense, then bank feed
   imports the same transaction
2. **Invoice + payment confusion**: User creates an invoice, then also
   records it as a sale
3. **Bill + expense confusion**: User records a bill, then also records
   the payment as a separate expense
4. **Recurring transactions**: Monthly rent/subscription recorded twice
