---
name: journal-entries
description: Manage the general ledger — record journal entries, adjusting entries, transfers, and corrections. Use when the user mentions journal entries, JE, transfers, reclassifying, or reversing entries.
---

# Journal Entries Skill

Manage the general ledger directly. Record standard journal entries, handle inter-account transfers, and perform period corrections using reversing entries.

## Trigger

Activate when the user wants to:
- Make a journal entry or adjusting entry
- Transfer funds between accounts
- Reclassify a transaction
- Correct or reverse a transaction from a closed period
- Book accruals or depreciation directly

## Workflow: Record a Journal Entry

1. **Lookup** — `qbMasterData` for account IDs, class/location IDs if needed
2. **Duplicate Check** — `qbFetchTransactions(transactionType="JournalEntry")` matching date and total amount
3. **Verify Balance** — Ensure total Debits exactly equal total Credits
4. **Confirm** — Show the proposed entry: date, accounts, debits, credits, and memo
5. **Record** — `qbJournalEntry` with the balanced lines
6. **Attach** — `qbAttachFile` (entityType = "JournalEntry") — supporting schedule, approval email, or source doc from portal, local file, drive, or user upload; preferred for audit-ready books
7. **Rule** — Include a descriptive `memo` detailing the purpose of the entry

## Workflow: Record a Transfer

When moving money between two internal accounts (e.g., Checking to Savings):

1. **Lookup** — `qbMasterData` for source and destination account IDs
2. **Duplicate Check** — `qbFetchTransactions(transactionType="Transfer")` matching date and amount
3. **Record** — `qbTransfer` with `fromAccountId`, `toAccountId`, `amount`, and `date`
4. **Note** — Do not use a Journal Entry for a simple transfer between bank accounts; use a Transfer.

## Workflow: Corrections & Reversals

When correcting errors in the general ledger:

### Same Period Correction (Voiding)
If the error is in the current, open period:
1. `qbFetchTransactions` — find the incorrect transaction
2. `qbVoidTransaction` — void it (preserves the audit trail, unlike delete)
3. Record the correct transaction using the appropriate skill

### Prior Period Correction (Reversing Entry)
If the error is in a closed period, do not modify the original transaction:
1. Create a Journal Entry that exactly reverses the original entry (swap debits and credits)
2. Record the reversing entry in the current, open period
3. Add a clear memo: "Correction of [original transaction ref] from [date]"
4. Record the correct entry in the current period if applicable

## Safety Checklist

- [ ] `qbMasterData` lookup completed for valid account IDs
- [ ] Journal Entry debits = credits exactly
- [ ] Duplicate check completed before recording
- [ ] Appropriate memo included for audit trail
- [ ] User confirmation obtained before creating
