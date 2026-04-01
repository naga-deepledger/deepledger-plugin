# /find

Search for transactions, vendors, customers, or accounts in QuickBooks.

## Usage
```
/find <search query>
```

## Examples
```
/find all bills from Acme Corp this month
/find expenses over $1000 in March
/find unpaid invoices for Customer ABC
/find vendor named "Office Depot"
/find checking account balance
/find all transactions last week
```

## Behavior

1. Parse the search query to determine:
   - **Entity type**: transaction, vendor, customer, account, or item
   - **Filters**: date range, amount range, vendor/customer, status (paid/unpaid/outstanding)
2. Route to the right tool:
   - Transactions → `qbFetchTransactions` with appropriate filters
   - Vendors/Customers/Accounts/Items → `qbMasterData` with name filter
   - Outstanding bills → `qbFetchTransactions(type="Bill", outstandingOnly=true)`
   - Outstanding invoices → `qbFetchTransactions(type="Invoice", outstandingOnly=true)`
3. Present results in a clear table format
4. If too many results, suggest narrowing the search

## Smart Defaults
- If no date range specified, default to current month
- If searching for a vendor/customer, also show recent transactions with them
- If searching for an account, show the current balance
