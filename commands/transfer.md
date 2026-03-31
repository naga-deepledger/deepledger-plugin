---
description: Transfer money between bank accounts or pay a credit card
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: <description of transfer>
---

Record a bank transfer based on the user's description: "$ARGUMENTS"

Use when moving money between the company's own accounts (bank-to-bank,
bank-to-credit card payment, savings-to-checking, etc.).

Steps:
1. Parse the description to determine:
   - Source account (money coming FROM)
   - Destination account (money going TO)
   - Amount
   - Date (default today)
2. Look up both accounts via qbMasterData (type: "Account")
   - Match by name — auto-select if obvious
   - If only one account mentioned, flag for review for the other
   - Both must be Bank, CreditCard, or OtherCurrentAsset type
3. Check for duplicates via qbFetchTransactions
4. If all data is clear, execute immediately via qbTransfer:
   - fromAccountId: source account ID
   - toAccountId: destination account ID
   - amount: transfer amount
   - txnDate: YYYY-MM-DD
   - memo: auto-generated "Transfer: [source] → [destination]"
5. Report success with transaction ID

**Critical:** Transfers are NOT expenses. Moving money between your own
accounts doesn't change total assets (just shifts between accounts).
Credit card payments reduce both cash and credit card liability.

Common patterns:
- "Transfer $5,000 from savings to checking" → fromAccountId=Savings, toAccountId=Checking
- "Pay off credit card" → fromAccountId=Checking, toAccountId=CreditCard
- "Move $10K to operating account" → identify source, toAccountId=operating account
