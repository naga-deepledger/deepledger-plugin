# QuickBooks Online API Capabilities Matrix

## Purpose

This document maps which QuickBooks features are available via the REST API
vs. which require browser automation (QBO UI). Use this to route tasks
to the correct integration channel.

## Transaction Types — API Support

| Entity | API Support | CRUD | Notes |
|--------|------------|------|-------|
| Purchase (Expense) | Full | CRUD | Includes Check, CreditCard, Cash types |
| Bill | Full | CRUD | Vendor bills/invoices |
| BillPayment | Full | CRUD | Check and CreditCard payment types |
| Invoice | Full | CRUD | Customer invoices |
| Payment (ReceivePayment) | Full | CRUD | Customer payments on invoices |
| SalesReceipt | Full | CRUD | Cash/card sales |
| CreditMemo | Full | CRUD | Customer credit memos |
| VendorCredit | Full | CRUD | Vendor credits |
| RefundReceipt | Full | CRUD | Customer refunds |
| Deposit | Full | CRUD | Bank deposits |
| JournalEntry | Full | CRUD | Adjusting entries |
| Transfer | Full | CRUD | Native bank-to-bank transfers |
| Estimate | Full | CRUD | Quotes/proposals, convertible to Invoice |
| PurchaseOrder | Full | CRUD | Vendor POs, convertible to Bill |
| TimeActivity | Full | CRUD | Time tracking entries |

## Master Data — API Support

| Entity | API Support | CRUD | Notes |
|--------|------------|------|-------|
| Customer | Full | CRUD | |
| Vendor | Full | CRUD | |
| Account (CoA) | Full | CRUD | |
| Item (Product/Service) | Full | CRUD | |
| Class | Full | CRUD | Tracking categories |
| Department | Full | CRUD | Location/department tracking |
| Term (Payment Terms) | Full | CRUD | Net 30, etc. |
| PaymentMethod | Full | CRUD | |
| TaxCode | Partial | Read | Read-only; creation requires Tax API |
| TaxRate | Partial | Read | Read-only |
| Attachable | Full | CRUD | File attachments on transactions |
| CustomField | Partial | Read | Can read values; field definition is UI-only |

## Reports — API Support

| Report | API Support | Notes |
|--------|------------|-------|
| ProfitAndLoss | Full | Supports date ranges, comparison |
| BalanceSheet | Full | Supports as-of date |
| CashFlow | Full | Statement of cash flows |
| TrialBalance | Full | |
| AgedReceivables | Full | Summary and detail versions |
| AgedPayables | Full | Summary and detail versions |
| GeneralLedger | Full | |
| TransactionList | Full | Filterable by type, date, entity |
| CustomerIncome | Full | Revenue by customer |
| VendorExpenses | Full | Expenses by vendor |

## Features — NO API (Browser Only)

| Feature | Why No API | Workaround |
|---------|-----------|------------|
| **Bank Reconciliation** | Intuit doesn't expose reconciliation endpoints | Browser automation via `/reconcile` |
| **Bank Feeds** | Bank connections managed exclusively in UI | Browser automation for matching |
| **Bank Rules** | Auto-categorization rules are UI-only | Browser automation via `/bank-rules` |
| **Recurring Transactions** | No CRUD for scheduled/recurring templates | Browser automation via `/recurring` |
| **Budgets** | No read or write API for QBO budgets | Historical baseline via `/budget` command; or browser for native budgets |
| **Audit Log** | No API access to audit trail | Browser navigation to Settings → Audit Log |
| **1099 Tracking** | Managed through Intuit's 1099 service | Browser navigation to Expenses → Vendors → 1099s |
| **Payroll** | Separate Intuit Payroll API (different auth) | Not covered; requires separate integration |
| **Multi-Currency Setup** | Enabling multi-currency is UI-only | Once enabled in UI, API supports foreign currency transactions |

## Rate Limits

| Limit | Value |
|-------|-------|
| Requests per minute per company | 500 |
| Concurrent requests per company | 10 |
| Batch operations per request | 30 |

## MCP Tool Mapping

Current MCP tools available via DeepLedger:

| MCP Tool | QBO Entity | Available |
|----------|-----------|-----------|
| qbExpense | Purchase | Yes |
| qbBill | Bill | Yes |
| qbBillPayment | BillPayment | Yes |
| qbInvoice | Invoice | Yes |
| qbReceivePayment | Payment | Yes |
| qbSalesReceipt | SalesReceipt | Yes |
| qbDeposit | Deposit | Yes |
| qbRefundReceipt | RefundReceipt | Yes |
| qbCredit | CreditMemo / VendorCredit | Yes |
| qbJournalEntry | JournalEntry | Yes |
| qbVoidTransaction | (various) | Yes |
| qbMasterData | (various entities) | Yes |
| qbFetchTransactions | Query | Yes |
| qbReports | Reports | Yes |
| qbGetUploadUrl | Attachable | Yes |

### MCP Tools NOT Yet Available (API supports them)

These QBO API entities are supported but don't have dedicated MCP tools yet.
Workarounds are documented in the relevant command files.

| QBO Entity | Workaround |
|-----------|------------|
| Transfer | Use JournalEntry (debit/credit pattern) |
| Estimate | Use draft Invoice with "ESTIMATE" memo |
| PurchaseOrder | Record Bill when goods arrive |
| TimeActivity | Not yet covered |

Consider requesting these as new MCP tools from DeepLedger for cleaner
integration.
