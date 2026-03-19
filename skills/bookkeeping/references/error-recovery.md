# Error Recovery & Edge Cases

## API Error Handling

### Common QB API Errors and Responses

| Error | Likely Cause | Recovery |
|-------|-------------|----------|
| "Business Validation Error" | Invalid field value (wrong account type, missing required field) | Check field values against qbMasterData, fix and retry |
| "Stale Object" / SyncToken mismatch | Transaction was modified since we fetched it | Re-fetch transaction to get current SyncToken, then retry |
| "Duplicate Document Number" | Invoice/bill number already exists | Append suffix (-2, -3) or let QB auto-generate |
| "Object Not Found" | Referenced entity (vendor, customer, account) doesn't exist | Re-lookup via qbMasterData, verify ID, ask user if entity changed |
| "Permission Denied" | User's QB account lacks write access | Inform user: "Your QuickBooks connection doesn't have write permissions for this operation. Check your DeepLedger connection settings." |
| Rate limit / timeout | Too many requests or slow response | Wait 5 seconds, retry once. If still failing, inform user. |

### Recovery Workflow

1. **On any write failure:**
   - DO NOT silently retry with different data
   - Report the error clearly: "QuickBooks returned an error: [error message]"
   - Explain what likely went wrong in plain language
   - Suggest specific fix
   - Offer to retry with the correction

2. **On read failure (qbReports, qbFetchTransactions, qbMasterData):**
   - Retry once silently
   - If still failing, inform user and suggest: "QuickBooks may be temporarily
     unavailable. Try again in a few minutes."
   - Do NOT present partial data as if it's complete

3. **On entity not found during workflow:**
   - If vendor/customer lookup returns zero results:
     "I couldn't find '[name]' in QuickBooks. Would you like me to:
     (a) Search again with a different name, or
     (b) Create '[name]' as a new [vendor/customer]?"
   - If account lookup returns zero results:
     Show available accounts from the relevant category and ask user to pick

## Edge Cases

### Negative Amounts
- Vendor credits: Positive amount (qbCredit handles the direction)
- Customer refunds: Positive amount (qbRefundReceipt handles the direction)
- NEVER record a negative-amount expense or invoice — use the correct
  transaction type (credit memo, refund, void) instead
- If user says "we got $500 back from Amazon" → this is a vendor credit, not
  a negative expense

### Zero-Amount Transactions
- Generally don't create $0 transactions — inform user it's unusual
- Exception: $0 invoice for tracking/record purposes (e.g., warranty service)
- Exception: Journal entry where debits = credits = $0 (no-op, skip it)

### Foreign Currency
- QB Online supports multi-currency but it must be enabled at company level
- If user mentions a foreign currency amount:
  - Check if multi-currency is enabled (visible in qbMasterData account details)
  - If enabled: ask for exchange rate or let QB use the day's rate
  - If not enabled: inform user amounts must be in home currency and ask them
    to convert

### Very Large Transactions (>$100K)
- Valid for many businesses — don't block
- Add extra emphasis in the confirmation: "Please confirm this $X transaction"
- Always show the amount prominently in the proposal

### Partial Payments
- When a customer pays less than the full invoice amount:
  - Record the partial payment via qbReceivePayment linked to the invoice
  - QB automatically tracks the remaining balance
  - Note in confirmation: "Partial payment of $X applied. Remaining balance: $Y"
- When paying a bill partially:
  - Use qbBillPayment with partial amount linked to the bill
  - Note remaining amount

### Backdated Transactions
- Transactions dated in a prior closed period:
  - Warn: "This date falls in [month] which may already be closed. Recording
    this will change prior period financials. Proceed?"
  - If >90 days old: stronger warning about audit implications
- Transactions dated in the future:
  - Acceptable for invoices (due dates) and scheduled bills
  - Unusual for expenses — ask: "This date is in the future. Is that correct?"

### Duplicate Vendor/Customer Names
- "Office Depot" vs "Office Depot Inc" vs "Office Depot #1234"
  - These are likely the same entity → suggest merging (user must do in QB UI)
  - For now, pick the one with most transaction history
  - Note: "Multiple similar vendors found. Using '[name]' (most active).
    Consider merging duplicates in QuickBooks."

### Multi-Line Transactions with Mixed Categories
- "Paid Costco $500 — $300 for office supplies and $200 for employee lunch"
  - Record as single expense with two line items:
    - Line 1: $300, Office Supplies account
    - Line 2: $200, Meals & Entertainment account
  - Each line gets its own account categorization

### Voiding vs Deleting
- **Void**: Transaction stays in records with $0 amount — audit trail preserved
- **Delete**: Transaction removed entirely — no audit trail
- **Default to void** unless user specifically asks to delete
- Always warn: "Voiding is preferred over deleting for audit trail purposes.
  Shall I void this transaction?"

### Undeposited Funds Buildup
- If Undeposited Funds balance grows beyond a reasonable amount (>$5K or >30 days):
  - During health checks and close-books, flag it
  - "Undeposited Funds balance is $X. This typically means received payments
    haven't been deposited yet. Would you like to create a bank deposit?"

### Credit Card Payments Misrecorded as Expenses
- Common mistake: recording a credit card payment as an expense
- If user says "paid credit card bill" or "credit card payment":
  - This is a TRANSFER (reduces CC liability + reduces bank balance)
  - NOT an expense (doesn't affect P&L)
  - Use the transfer workflow (Journal Entry pattern)
  - If they insist on expense: warn them it will double-count
