---
description: Manage recurring/scheduled transactions
allowed-tools: ["mcp__playwright__*", "mcp__plugin_deepledger_deepledger__*"]
argument-hint: <create|list|edit|delete> [details]
---

Manage recurring transactions in QuickBooks: "$ARGUMENTS"

**API Status:** Recurring transactions support **list, read, create, and delete
via the `qbRecurringTransaction` API tool**. Browser automation is only needed
for **editing** an existing recurring template (the QuickBooks API does not
support updates to existing recurring transactions — delete and recreate instead).

**Prerequisites for browser operations:** User must be logged into QuickBooks
Online in their browser.

## Operations

### List Recurring Transactions (API)
1. Call `qbRecurringTransaction(operation: "list")` — returns all recurring templates
2. Optionally filter by type: `qbRecurringTransaction(operation: "list", filterType: "Bill")`
3. Present a table: Type, Template Name, Vendor/Customer, Amount, Frequency, Next Date

### Create Recurring Transaction (API)
1. Determine transaction type from user's description:
   - "recurring bill" → Bill
   - "recurring expense" → Purchase
   - "recurring invoice" → Invoice
   - "monthly rent" → Bill or Purchase (infer from context)
2. Look up vendor/customer and account via `qbMasterData` to get IDs
3. Call `qbRecurringTransaction(operation: "create", createTransactionType: <type>, recurringInfo: { name, recurType, scheduleInfo }, vendorId/customerId, lines)`
   - recurType: "Automated" (auto-creates), "Reminder" (notifies), or "Unscheduled" (template only)
   - scheduleInfo: intervalType (Daily/Weekly/Monthly/Yearly), numInterval, dayOfMonth, startDate
4. Confirm creation with the template ID

### Edit Recurring Transaction (Browser — API does not support update)
1. Navigate to Settings → Recurring Transactions
2. Find the template by name or description
3. Click to edit and make the requested changes
4. Take screenshot for confirmation
5. On approval, save

### Delete Recurring Transaction (API)
1. Call `qbRecurringTransaction(operation: "read", recurringTransactionId: <id>)` to get syncToken
2. Deleting a recurring template is destructive — escalate via contactHuman
   for confirmation before proceeding
3. On explicit human confirmation, call `qbRecurringTransaction(operation: "delete", recurringTransactionId, syncToken, transactionType)`
4. Log action via agentLog and confirm deletion

## Common Recurring Setups

| Description | Type | Frequency | Suggested Template Name |
|-------------|------|-----------|------------------------|
| Monthly rent | Expense/Bill | Monthly | "Monthly Rent - [Landlord]" |
| Software subscription | Expense | Monthly | "[Software] Subscription" |
| Quarterly insurance | Expense/Bill | Quarterly | "[Carrier] Insurance Premium" |
| Monthly invoicing | Invoice | Monthly | "Monthly Retainer - [Client]" |
| Weekly cleaning | Expense | Weekly | "Weekly Cleaning - [Vendor]" |
| Annual domain renewal | Expense | Yearly | "Domain Renewal - [Provider]" |

## Hybrid Approach

For detecting recurring patterns in existing transactions (without creating
formal QB recurring templates), use the API-based detection in
`references/recurring-transactions.md`. The browser command is only needed
for creating/managing formal QBO recurring transaction templates.
