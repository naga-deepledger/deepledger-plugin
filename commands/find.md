---
description: Search transactions by vendor, customer, date, or amount
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: <search criteria>
---

Search for transactions matching: "$ARGUMENTS"

Operate autonomously — interpret the search criteria intelligently and
return results without asking clarifying questions unless the search is
too vague to execute.

Steps:
1. Parse search criteria from the user's input (infer all of these):
   - Vendor or customer name
   - Date or date range (default: last 30 days if not specified)
   - Amount or amount range
   - Transaction type (expense, bill, invoice, etc.)
   - Account or category
2. Use qbFetchTransactions with appropriate filters
   - If a vendor/customer name is given, look up the entity first via
     qbMasterData to get the correct ID — auto-select single/obvious match
3. If too many results (>20), automatically narrow by the most relevant
   filters rather than asking the user — sort by date (most recent first)
   and show the top results with a note about total count
4. Display results in a clear table format:
   - Date
   - Type (Bill, Expense, Invoice, etc.)
   - Vendor/Customer
   - Amount
   - Account
   - Transaction ID
5. After showing results, suggest relevant next actions based on what was
   found (e.g., "Would you like to void any of these?" or "I can pull
   details on any specific transaction")
6. Only ask for clarification if the search criteria is genuinely too vague
   to execute (e.g., user just says "find" with no criteria)
