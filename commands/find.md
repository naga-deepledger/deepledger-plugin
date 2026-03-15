---
description: Search transactions by vendor, customer, date, or amount
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: <search criteria>
---

Search for transactions matching: "$ARGUMENTS"

Steps:
1. Parse search criteria from the user's input:
   - Vendor or customer name
   - Date or date range
   - Amount or amount range
   - Transaction type (expense, bill, invoice, etc.)
   - Account or category
2. Use qbFetchTransactions with appropriate filters
3. If too many results, narrow by asking the user for more specific criteria
4. Display results in a clear table format:
   - Date
   - Type (Bill, Expense, Invoice, etc.)
   - Vendor/Customer
   - Amount
   - Account
   - Transaction ID
5. If user wants to act on a result (void, pay, edit), guide them to the
   appropriate next step
