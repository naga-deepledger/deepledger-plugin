# Duplicate Detection — Methodology and Procedures

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

## How to Check

### Step 1: Query Existing Transactions

Use qbFetchTransactions with these filters:
- **Vendor/Customer**: Same entity name
- **Date range**: ±3 days from the intended transaction date
- **Transaction type**: Same type if possible

### Step 2: Compare Results

A potential duplicate exists if ALL of these match:
- Same vendor or customer (exact or fuzzy name match)
- Same date (±1 day for potential timezone differences)
- Same total amount (±5% for rounding/tax differences)

### Step 3: Present to User

If potential duplicates found, show them in a table:
```
Existing transactions that may be duplicates:
| Date       | Type    | Vendor      | Amount  | ID    |
|------------|---------|-------------|---------|-------|
| 2026-03-13 | Expense | Staples     | $150.00 | #1234 |
```

Then ask: "A similar transaction already exists. Is this a new,
separate transaction?"

### Step 4: Proceed or Abort

- If user confirms it's new → proceed with creation
- If user says it's a duplicate → abort and reference the existing one

## Fuzzy Name Matching

Vendor/customer names may not match exactly:
- "Office Depot" vs "Office Depot Inc"
- "Amazon" vs "Amazon.com" vs "AMZN"
- "John's Plumbing" vs "Johns Plumbing LLC"

When searching, use partial name matches. If the vendor/customer lookup
returns multiple similar results, list them all.

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
