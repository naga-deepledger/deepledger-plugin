---
description: Manage recurring/scheduled transactions
allowed-tools: ["mcp__browser__*", "mcp__plugin_deepledger_deepledger__*"]
argument-hint: <create|list|edit|delete> [details]
---

Manage recurring transactions in QuickBooks: "$ARGUMENTS"

**Important:** Recurring transactions are NOT available via the QuickBooks API.
This command uses browser automation to interact with the QBO UI directly.

**Prerequisites:** User must be logged into QuickBooks Online in their browser.

## Operations

### List Recurring Transactions
1. Navigate to Settings (gear icon) → Recurring Transactions
2. Take screenshot of the list
3. Present a table: Type, Template Name, Vendor/Customer, Amount, Frequency, Next Date

### Create Recurring Transaction
1. Determine transaction type from user's description:
   - "recurring bill" → Bill
   - "recurring expense" → Expense (Check/Credit Card)
   - "recurring invoice" → Invoice
   - "monthly rent" → Expense or Bill (infer from context)
2. Look up vendor/customer and account via API (qbMasterData) to verify they exist
3. Navigate to Settings → Recurring Transactions → New
4. Select transaction type
5. Fill in template details:
   - Template name (auto-generate from vendor + description)
   - Type: Scheduled (auto-create), Reminder, or Unscheduled
   - Vendor/customer, account, amount, memo
   - Interval: Daily/Weekly/Monthly/Yearly
   - Start date, end date (or no end)
6. Take screenshot for confirmation
7. On user approval, click Save template

### Edit Recurring Transaction
1. Navigate to Settings → Recurring Transactions
2. Find the template by name or description
3. Click to edit
4. Make the requested changes
5. Take screenshot for confirmation
6. On approval, save

### Delete Recurring Transaction
1. Navigate to Settings → Recurring Transactions
2. Find the template
3. Show current details to user
4. On explicit confirmation, delete
5. Confirm deletion

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
