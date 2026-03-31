---
description: Reconcile a bank or credit card account
allowed-tools: ["mcp__playwright__*", "mcp__plugin_deepledger_deepledger__*"]
argument-hint: <account name> [statement date] [ending balance]
---

Reconcile a bank or credit card account: "$ARGUMENTS"

**Important:** Bank reconciliation is NOT available via the QuickBooks API.
This command uses browser automation to interact with the QBO UI directly.

**Prerequisites:**
- User must be logged into QuickBooks Online in their browser
- User needs their bank/credit card statement with ending balance and date

Steps:

**Phase 1: Preparation (via API)**
1. Look up the account to reconcile via qbMasterData
   - If account name is ambiguous, show options and ask
2. Pull recent transactions for context via qbFetchTransactions
3. Pull the balance sheet to see current book balance

**Phase 2: Get Statement Info**
If not provided in the arguments, ask the user for:
- Statement ending date
- Statement ending balance
- (Optional) Statement beginning balance for verification

**Phase 3: Browser Automation**
4. Navigate to QBO reconciliation page (Banking → Reconcile or
   Accounting → Reconcile)
5. Select the account from the dropdown
6. Enter statement date and ending balance
7. Click "Start reconciling"
8. Take a screenshot of the reconciliation screen showing all transactions

**Phase 4: Matching**
9. Present the list of uncleared transactions to the user
10. Ask the user which transactions appear on their statement
    - Or if they can share/describe their statement, help match automatically
11. Check/uncheck transactions as directed
12. Monitor the "Difference" field — goal is $0

**Phase 5: Completion**
13. When difference = $0, click "Finish now" to complete reconciliation
14. Take screenshot of completed reconciliation
15. Report: "Reconciliation complete for [account] through [date]. [X] transactions cleared, ending balance $Y."

**If difference ≠ $0:**
- Show the discrepancy amount
- Help investigate: check for missing transactions, timing differences,
  bank fees not yet recorded
- Offer to record adjusting entries if needed
- User can choose to save for later ("Save for later" button)

**Common issues:**
- Opening balance doesn't match → may need to reconcile prior months first
- Transactions in QB not on statement → haven't cleared the bank yet (leave unchecked)
- Transactions on statement not in QB → need to be recorded first (offer to record via API)
