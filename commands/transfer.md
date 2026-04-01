# /transfer

Transfer money between your own bank accounts in QuickBooks.

## Usage
```
/transfer <amount> from <source account> to <destination account>
```

## Examples
```
/transfer $5,000 from checking to savings
/transfer 2500 from business checking to payroll account
/transfer $10,000 from savings to operating account
```

## Behavior

Use the **Accountant** agent with the **Bookkeeping** skill.

1. Parse the amount, source account, and destination account
2. Look up both account IDs: `qbMasterData(entityType="Account", filter="Bank")`
3. Duplicate check: `qbFetchTransactions(type="Transfer")` for same amount + date
4. Show confirmation: "Transfer $X from [Source] to [Destination] on [Date]?"
5. Record: `qbTransfer` with fromAccountId, toAccountId, amount, date
6. Report the result with transaction ID

## Validation
- Both accounts must be Bank type
- Source and destination must be different accounts
- Amount must be positive
