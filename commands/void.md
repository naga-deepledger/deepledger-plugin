# /void

Void a transaction in QuickBooks Online. Preserves the audit trail — the transaction remains visible but is marked void with a $0 amount.

## Usage
```
/void <transaction description or ID>
```

## Examples
```
/void the bill payment to Acme Corp from last Tuesday
/void invoice #1042 for Customer ABC
/void the sales receipt for $850 on March 15
/void transaction ID 123
```

## Behavior

Use the **Accountant** agent.

### Supported Transaction Types

`qbVoidTransaction` supports voiding the following types:
- `BillPayment`
- `Invoice`
- `Payment` (customer payment / ReceivePayment)
- `SalesReceipt`
- `CreditMemo`
- `Purchase` (Expense)
- `RefundReceipt`
- `Transfer`

### Unsupported Transaction Types

The following **cannot be voided via tools** — they must be handled directly in QuickBooks Online by the user or CPA:
- `Bill`
- `JournalEntry`
- `Deposit`
- `Expense` recorded without a vendor (unlinked)
- `VendorCredit`

If the user requests voiding an unsupported type, explain this limitation and advise them to void it manually in QB.

### Workflow

1. **Find the transaction** — `qbFetchTransactions` using the description, vendor/customer name, amount, date, or transaction ID provided by the user
2. **Display the transaction** — show the user: type, date, amount, vendor/customer, and current status
3. **Verify it is voidable** — confirm the transaction type is in the supported list above
4. **Confirm intent** — ask the user to confirm they want to void this specific transaction and explain the effect (zeroes the amount, preserves the record)
5. **Void** — `qbVoidTransaction` with the transaction ID and type
6. **Report** — confirm the void was successful and show the resulting $0 transaction

## Safety
- NEVER void without first fetching and displaying the transaction to the user
- NEVER void without explicit user confirmation
- If multiple transactions match the description, show all matches and ask the user to select one — do not guess
- Voiding is irreversible via tools — make sure the user understands before proceeding
- Prefer voiding over deleting — voiding preserves the audit trail
