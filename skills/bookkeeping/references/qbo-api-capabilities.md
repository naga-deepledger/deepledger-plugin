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
| RecurringTransaction | **Read-only** | R_ | Query and read only. **Cannot create, update, or delete via API** |
| Budget | **Read-only** | R_ | Query and read only. **Cannot create, update, or delete via API** |

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
| **Recurring Txn Create/Edit** | Read-only API (can query/read but NOT create/update/delete) | Browser automation via `/recurring` to create/edit. Use API to LIST existing recurring transactions |
| **Budget Create/Edit** | Read-only API (can query/read but NOT create/update/delete) | Use API to READ existing budgets for `/budget` comparisons. Browser to create/edit |
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

### MCP Tools NOT Yet Available (API supports them)

These QBO API entities have full API support but no dedicated MCP tool yet.

| QBO Entity | Full API Support | Workaround |
|-----------|-----------------|------------|
| Transfer | CRUD | Use JournalEntry (debit/credit pattern) |
| Estimate | CRUD | Use draft Invoice with "ESTIMATE" memo |
| PurchaseOrder | CRUD | Record Bill when goods arrive |
| TimeActivity | CRUD | Not yet covered |
| InventoryAdjustment | Create/Update | Not yet covered |
| RecurringTransaction | **Read-only** | Read via API; create/edit via browser |
| Budget | **Read-only** | Read via API for comparisons; create via browser |

**Action items for DeepLedger MCP team:**
1. Add `qbTransfer` tool — native Transfer entity is cleaner than JE workaround
2. Add `qbEstimate` tool — full CRUD supported, common workflow
3. Add `qbPurchaseOrder` tool — full CRUD, Plus/Advanced plans
4. Add `qbTimeActivity` tool — enables time-based billing workflow
5. Add `qbRecurringTransaction` read tool — list scheduled transactions
6. Add `qbBudget` read tool — read budgets for comparison reports
