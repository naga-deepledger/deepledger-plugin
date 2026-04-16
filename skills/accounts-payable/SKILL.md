---
name: accounts-payable
description: Manage accounts payable — enter bills, pay vendors, apply vendor credits, monitor aging, and track outstanding obligations. Use when the user mentions bills, vendor payments, AP aging, bill pay, or outstanding payables.
---

# Accounts Payable Skill

Manage the full AP lifecycle: entering vendor bills, scheduling and making payments, applying vendor credits, and monitoring what you owe.

## Trigger

Activate when the user wants to:
- Enter a vendor bill
- Pay a vendor bill
- Apply a vendor credit
- Check outstanding or overdue payables
- Review vendor spending
- Process a batch of bill payments
- Track payment terms and due dates

## The AP Flow

```
Bill → BillPayment → Bank Account
```

- `qbBill` creates the payable (AP goes up, no cash out yet)
- `qbBillPayment` pays the bill from a bank or credit card account (AP goes down, cash goes down)

For immediate-pay purchases with no bill: `qbExpense` records the payment directly.

## When to Use Bill vs Expense

| Situation | Tool | Why |
|-----------|------|-----|
| Vendor sends an invoice to pay later | `qbBill` | Creates AP, tracks the obligation |
| Paid vendor immediately (card swipe, ACH) | `qbExpense` | No AP needed, money already left |
| Paying a previously entered bill | `qbBillPayment` | Clears the AP, records the payment |

**Key rule**: If a `qbBill` already exists for this vendor+amount+date, use `qbBillPayment` — never create a duplicate `qbExpense`.

## Workflow: Enter a Bill

1. **Lookup** — `qbMasterData` for vendor ID and expense account IDs
2. **Check memory** — `agentMemory` for vendor-to-account mapping and typical amounts
3. **Duplicate check** — `qbFetchTransactions(transactionType="Bill", outstandingOnly=true, entityId=vendorId)` to verify no existing bill for same vendor+amount
4. **Build bill** — Set `vendorId`, `txnDate`, `dueDate`, `lines` with account/amount
5. **Confirm** — Show: vendor, total, due date, expense categories
6. **Record** — `qbBill`
7. **Attach** — If source document available, `qbAttachFile` (entityType = "Bill")
8. **Learn** — `agentMemory` upvote vendor mapping

## Workflow: Pay a Bill

1. **Find outstanding bills** — `qbFetchTransactions(transactionType="Bill", outstandingOnly=true, entityId=vendorId)`
2. **Select bills** — Identify which bill(s) to pay (by amount, date, or vendor)
3. **Choose payment account** — `qbMasterData` for Bank/CC account IDs
4. **Record** — `qbBillPayment` with `vendorId`, `paymentDate`, `bankAccountId`, `bills` array (billId + amount per bill)
5. **Partial payments** — Pay less than the full bill amount; the bill remains partially outstanding
6. **Multiple bills** — Pay several bills to one vendor in a single BillPayment using the `bills` array
7. **Confirm** — Show: vendor, bills being paid, total, payment account

## Workflow: Record an Expense (Immediate Payment)

1. **Check for existing bill** — `qbFetchTransactions(transactionType="Bill", outstandingOnly=true, entityId=vendorId)` — if a matching bill exists, use `qbBillPayment` instead
2. **Lookup** — `qbMasterData` for vendor ID, source account (bank/CC), category account
3. **Duplicate check** — `qbFetchTransactions(transactionType="Purchase", entityId=vendorId)` matching amount and date
4. **Record** — `qbExpense` with `paymentType`, `accountId` (source), `vendorId`, `lines` (category accounts)
5. **Rule** — Source account (where money comes from) must differ from line accountId (what it was spent on)

## Workflow: Apply Vendor Credit

When a vendor issues a credit or refund:

1. **Create credit** — `qbCredit(creditType="vendor")` with `vendorId`, `txnDate`, `lines` (account + amount)
2. **Apply to payment** — In `qbBillPayment`, include the VendorCredit in the `bills` array with `txnType: "VendorCredit"`
3. Use for: vendor refunds, billing adjustments, returned goods

## Workflow: AP Aging Review

1. **Pull aging** — `qbReports(reportType="AgedPayables")` for summary, `AgedPayablesDetail` for line items
2. **Calculate DPO** — Days Payable Outstanding = AP / (Annual COGS / 365)
3. **Bucket analysis** — Present in aging buckets: Current, 1-30, 31-60, 61-90, 90+
4. **Flag concerns**:
   - Bills past due → late payment penalties, vendor relationship risk
   - AP growing faster than revenue → cash flow pressure
   - Large upcoming payments → cash planning needed
5. **Recommendations** — Bills to prioritize for payment, vendors to negotiate terms with

## Workflow: Vendor Spending Analysis

1. **Pull report** — `qbReports(reportType="VendorExpenses")` for spending by vendor
2. **Identify** — Top vendors by total spend, fast-growing vendors, new vendors
3. **Compare** — Current period vs prior period
4. **Flag** — Vendor spend up 20%+ without corresponding revenue growth

## Safety Checklist

- [ ] `qbMasterData` lookup completed — valid vendor and account IDs
- [ ] Duplicate check — no existing bill or expense for same vendor+amount+date
- [ ] Check for outstanding bills before recording an expense
- [ ] Source account differs from category account on expense lines
- [ ] User confirmation before recording
- [ ] Agent memory updated after successful recording

## Common Mistakes to Avoid

- Recording an `qbExpense` when an outstanding `qbBill` already exists → use `qbBillPayment`
- Forgetting `vendorId` on expenses → breaks vendor spending reports and audit trail
- Source account same as category account → accounting error
- Creating a BillPayment without checking for outstanding bills first
- Wrong `entityType` when attaching documents: Expense = "Purchase" in the QB API
- Paying a bill from the wrong bank account
