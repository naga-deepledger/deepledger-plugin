# /attach

Attach a document (receipt, invoice, contract, remittance) to an existing QuickBooks transaction.

## Usage
```
/attach <transaction description> <document description or path>
```

## Examples
```
/attach the Acme Corp bill from March 15 — receipt is in the portal
/attach invoice #1042 — signed contract at /home/user/docs/contract.pdf
/attach the $500 Office Depot expense — I'll upload the receipt
/attach bill payment to AWS last Friday — bank confirmation from email
```

## Behavior

Use the **Accountant** agent.

### Supported Entity Types

`qbAttachFile` requires an `entityType` that matches the QuickBooks transaction type. The mapping is:

| Transaction | entityType |
|-------------|------------|
| Bill | `Bill` |
| Bill Payment | `BillPayment` |
| Expense (Purchase) | `Purchase` |
| Invoice | `Invoice` |
| Customer Payment (ReceivePayment) | `Payment` |
| Sales Receipt | `SalesReceipt` |
| Credit Memo | `CreditMemo` |
| Refund Receipt | `RefundReceipt` |

### Workflow

1. **Find the transaction** — `qbFetchTransactions` using the description, vendor/customer name, amount, or date provided by the user
2. **Display the transaction** — show the user: type, date, amount, vendor/customer, and transaction ID
3. **Confirm the match** — ask the user to confirm this is the correct transaction
4. **Determine the document source**:
   - Portal document → user specifies the document name or ID from the DeepLedger portal
   - Local file → user provides a file path
   - User upload → user will paste or drag the file into the conversation
5. **Determine entityType** — use the mapping table above based on the transaction type
6. **Attach** — `qbAttachFile(entityType=..., entityId=..., ...)` with the transaction ID
7. **Confirm** — report success and the document name attached

## When to Attach Documents

Attaching source documents is recommended (not required) for audit-ready books:
- **Bills** → vendor invoice PDF
- **Expenses** → receipt or credit card statement line
- **Invoices** → signed contract, PO, or supporting documentation
- **Payments** → bank remittance or payment confirmation
- **Bill Payments** → remittance advice or bank confirmation

## Safety
- Always find and display the target transaction before attaching — never attach blindly by description alone
- If multiple transactions match, show all and ask the user to select one
- Confirm the entityType matches the transaction type before calling `qbAttachFile`
