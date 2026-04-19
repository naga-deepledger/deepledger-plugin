# /ap

Manage accounts payable — enter bills, pay vendors, apply vendor credits, and review what you owe.

## Usage
```
/ap                          # Review outstanding bills and AP aging
/ap bill <description>       # Enter a new vendor bill
/ap pay <vendor>             # Pay outstanding bills for a vendor
/ap credit <vendor>          # Apply a vendor credit
/ap aging                    # AP aging report with DPO and recommendations
```

## Behavior

Use the **Accountant** agent with the **Accounts Payable** skill.

### /ap (no args) — Review Outstanding Bills
1. `qbReports(reportType="AgedPayables")` — pull summary aging
2. Present totals by bucket: Current, 1-30, 31-60, 61-90, 90+ days
3. Highlight past-due bills and any requiring immediate payment
4. Suggest next actions

### /ap bill — Enter a Vendor Bill
1. Parse vendor, amount, due date, and expense category from the description
2. `qbMasterData` — look up vendor ID and expense account IDs
3. `agentMemory(operation="read")` — check known vendor-to-account mapping
4. `qbFetchTransactions(transactionType="Bill", outstandingOnly=true, entityId=vendorId)` — duplicate check
5. Show confirmation: vendor, total, due date, expense categories
6. `qbBill` — record the bill
7. Prompt to attach the vendor invoice via `qbAttachFile(entityType="Bill")` — recommended for audit trail
8. `agentMemory` — upvote the vendor mapping

### /ap pay — Pay a Vendor Bill
1. `qbFetchTransactions(transactionType="Bill", outstandingOnly=true, entityId=vendorId)` — find open bills
2. Show the user which bills are outstanding; confirm which to pay and payment amount
3. `qbMasterData` — look up payment account (Bank or Credit Card)
4. Show confirmation: vendor, bills being paid, total, payment account
5. `qbBillPayment` with vendorId, paymentDate, bankAccountId, and bills array
6. Prompt to attach remittance advice via `qbAttachFile(entityType="BillPayment")` — recommended for audit trail

### /ap credit — Apply a Vendor Credit
1. `qbMasterData` — look up vendor ID and relevant account IDs
2. `qbCredit(creditType="vendor")` with vendorId, txnDate, and lines
3. To apply the credit against a payment: include the VendorCredit in the `bills` array of `qbBillPayment` with `txnType: "VendorCredit"`

### /ap aging — Aging Report with Analysis
Delegates to `/aging ap` — same report, same DPO calculation, same recommendations. Use `/aging ap` directly for aging analysis.

## Bill vs Expense Decision
| Situation | Tool |
|-----------|------|
| Vendor sends invoice — pay later | `qbBill` |
| Paid vendor immediately (card / ACH) | `qbExpense` |
| Paying a bill already entered | `qbBillPayment` |

**Key rule**: If an outstanding `qbBill` already exists for this vendor, use `qbBillPayment` — never create a duplicate `qbExpense`.

## Safety
- NEVER skip the duplicate check before entering a bill
- NEVER create a `qbExpense` when an outstanding `qbBill` exists for the same vendor and amount
- Source account must differ from category account on expense lines
- Always confirm with the user before recording any transaction
