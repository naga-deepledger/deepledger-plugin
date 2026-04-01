# /record

Record a financial transaction in QuickBooks Online.

## Usage
```
/record <description of transaction>
```

## Examples
```
/record Paid $500 to Office Depot for office supplies on 3/15 with company credit card
/record Bill from Acme Corp for $2,400 consulting services dated 3/20
/record Invoice Customer ABC $5,000 for web development project
/record Deposit $3,200 into checking from customer payment
/record Transfer $1,000 from checking to savings
/record Journal entry: reclassify $800 from Office Supplies to Marketing
```

## Behavior

Use the **Accountant** agent with the **Bookkeeping** skill.

1. Parse the user's description to determine transaction type, amount, vendor/customer, date, account, and memo
2. If any required info is missing, ask the user
3. Follow the transaction recording workflow from `getGuide(guideType="transaction_recording")`:
   - **Identify type** → Expense, Bill, Invoice, Deposit, Payment, Transfer, or Journal Entry
   - **Lookup** → `qbMasterData` for vendor/customer and account IDs
   - **Check memory** → `agentMemory` for known vendor-to-account mappings
   - **Duplicate check** → `qbFetchTransactions` with vendor + date + amount
   - **Confirm** → Show the user what will be recorded and get confirmation
   - **Record** → Use the appropriate QB tool
   - **Learn** → `agentMemory` upvote or write the vendor-to-account mapping
4. Report the result with the QB transaction ID

## Safety
- NEVER skip the duplicate check
- NEVER record without user confirmation
- Source account must differ from category account
- Journal Entry debits must equal credits
