# QuickBooks Online API Capabilities Matrix

## Purpose

This document maps which QuickBooks features are available via the REST API
vs. which require browser automation (QBO UI). Use this to route tasks
to the correct integration channel.

Last updated: March 2026. Sources: Intuit Developer docs, API release notes.

## Transaction Types — API Support

| Entity | API Support | CRUD | Notes |
|--------|------------|------|-------|
| Purchase (Expense) | Full | CRUD | Includes Check, CreditCard, Cash types |
| Bill | Full | CRUD | Bill status ("For Review"/"Unpaid"/"Paid") is NOT settable via API |
| BillPayment | Full | CRUD | Payment method field not returned in response |
| Invoice | Full | CRUD | |
| Payment (ReceivePayment) | Full | CRUD | |
| SalesReceipt | Full | CRUD | |
| CreditMemo | Full | CRUD | |
| VendorCredit | Full | CRUD | |
| RefundReceipt | Full | CRUD | |
| Deposit | Full | CRUD | |
| JournalEntry | Full | CRUD | |
| Transfer | Full | CRUD | Native bank-to-bank: `FromAccountRef`, `ToAccountRef`, `Amount` |
| Estimate | Full | CRUD | Quotes/proposals, convertible to Invoice |
| PurchaseOrder | Full | CRUD | Plus/Advanced plans only, convertible to Bill |
| TimeActivity | Full | CRUD | Time tracking; Premium Time API for Gold/Platinum partners |
| InventoryAdjustment | Partial | CR_U | Create and update only. **Delete NOT supported** |
| RecurringTransaction | Partial | CRD | List, read, create, delete via API. **Update not supported** — edit via browser |
| Budget | Partial | CRD | List, read, create, delete via API. **Update not supported** — edit via browser |

## Master Data — API Support

| Entity | API Support | CRUD | Notes |
|--------|------------|------|-------|
| Customer | Full | CRUD | |
| Vendor | Full | CRUD | Vendor Notes field NOT accessible via API |
| Employee | Full | CRUD | Address/phone formatting enforced since Jan 2026 |
| Account (CoA) | Full | CRUD | Delete = soft delete (marks inactive). Creation limits by subscription |
| Item (Product/Service) | Full | CRUD | Desktop-created items cannot be modified via QBO API |
| Class | Full | CRUD | Plus plan: max 40 combined classes + locations. Must enable in UI first |
| Department (Location) | Full | CRUD | Same subscription limits as Class |
| Term (Payment Terms) | Full | CRUD | Net 30, etc. |
| PaymentMethod | Full | CRUD | |
| TaxCode | Partial | Read + Create | Read via query. Create via `TaxService` proxy. US: AST auto-calculates rates |
| TaxRate | Partial | Read + Create | Read via query. Create via `TaxService` proxy. US: largely automated |
| Attachable | Full | CRUD | Upload, download, link to transactions |
| CustomField | Premium | CRUD | New API (Dec 2025). Gold/Platinum partners only. Up to 12 fields |
| ExchangeRate | Partial | R_U | Read and update. Multi-currency must be enabled in UI first |

## Reports — API Support

20+ report types available. Cap of **400,000 cells per response**.

| Report | API Support | Notes |
|--------|------------|-------|
| ProfitAndLoss | Full | + ProfitAndLossDetail variant |
| BalanceSheet | Full | + BalanceSheetDetail variant |
| CashFlow | Full | Statement of cash flows |
| TrialBalance | Full | |
| GeneralLedger | Full | |
| TransactionList | Full | Filterable by type, date, entity |
| JournalReport | Full | |
| AgedReceivables | Full | Summary and detail versions |
| AgedPayables | Full | Summary and detail versions |
| CustomerIncome | Full | Revenue by customer |
| CustomerBalance | Full | + CustomerBalanceDetail |
| CustomerSales | Full | |
| VendorExpenses | Full | |
| VendorBalance | Full | + VendorBalanceDetail |
| ItemSales | Full | Sales by item/product |
| DepartmentSales | Full | Sales by location |
| ClassSales | Full | Sales by class |
| AccountListDetail | Full | Chart of accounts detail |

## Features — NO API (Browser Only)

| Feature | API Reality | Our Approach |
|---------|-----------|--------------|
| **Bank Reconciliation** | No API. Can query reconciliation *status* (Reconciled/Cleared/Uncleared) per transaction, but cannot perform reconciliation | Browser automation via `/reconcile` |
| **Bank Feeds** | No API. Cannot access "For Review" bank feed items. Intuit won't expose bank statement data for security/economic reasons | Browser automation for matching |
| **Bank Rules** | No API. Auto-categorization rules are UI-only | Browser automation via `/bank-rules` |
| **Recurring Txn Edit** | API supports list/read/create/delete. **Update not supported** | Use qbRecurringTransaction for list/read/create/delete. Browser for edit only |
| **Budget Edit** | API supports list/read/create/delete. **Update not supported** | Use qbBudget for list/read/create/delete. Browser for edit only |
| **Audit Log** | No API | Browser navigation to Settings → Audit Log |
| **1099 Filing** | Vendor1099 boolean settable via API, but form generation/filing is UI-only | Set 1099 flag via API; filing via browser |
| **Payroll** | Separate GraphQL API, partner-tier gated | Not covered; requires separate integration |
| **Multi-Currency Enable** | Toggle is UI-only. Once enabled, API supports foreign currency transactions | One-time UI setup, then API handles transactions |
| **Class/Location Enable** | Toggle is UI-only. Once enabled, full API CRUD | One-time UI setup, then API handles data |
| **Vendor Notes** | Field not exposed in API | Browser only |
| **Delete InventoryAdjustment** | Create/update only, no delete | Void via JE workaround, or browser |

## Rate Limits

| Limit | Value |
|-------|-------|
| Standard API calls | 500 requests/minute per company |
| Per-second throttle | 10 requests/second per company per app |
| Concurrent requests | 10 simultaneous per company per app |
| Batch endpoint | 120 requests/minute per company |
| Resource-intensive endpoints | 200 requests/minute |
| Report cell cap | 400,000 cells per response |
| Change Data Capture | Max 30 days lookback |

## Upcoming API Changes (2026)

| Change | Deadline | Impact |
|--------|----------|--------|
| Webhooks → CloudEvents format | May 15, 2026 | Must migrate webhook consumers |
| ID field no longer sortable | Jan 27, 2026 (done) | Query adjustments needed |
| ID field: only `=` and `IN` filters | Active | No `>`, `<`, `LIKE` on IDs |
| Tags fully removed | May 2028 | Migrate to Custom Fields |
| Refresh tokens start expiring | Feb 2027 | 5-year max validity |

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
| qbTransfer | Transfer | Yes |
| qbEstimate | Estimate | Yes |
| qbPurchaseOrder | PurchaseOrder | Yes |
| qbRecurringTransaction | RecurringTransaction | Yes (list/read/create/delete) |
| qbBudget | Budget | Yes (list/read/create/delete) |

### MCP Tools NOT Yet Available (API supports them)

| QBO Entity | API Support | Workaround |
|-----------|------------|------------|
| TimeActivity | Full CRUD | Not yet covered |
| InventoryAdjustment | Create/Update | Not yet covered |

**Action items for DeepLedger MCP team:**
1. Add `qbTimeActivity` tool — enables time-based billing workflow
2. Add `qbInventoryAdjustment` tool — create/update inventory adjustments
