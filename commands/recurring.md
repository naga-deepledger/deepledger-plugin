# /recurring

Manage recurring transactions in QuickBooks — list, create, pause, or resume automated entries.

## Usage
```
/recurring                     # List all active recurring transactions
/recurring list                # Same as above
/recurring create <description># Create a new recurring transaction
/recurring pause <name>        # Pause a recurring transaction
/recurring resume <name>       # Resume a paused recurring transaction
/recurring delete <name>       # Delete a recurring transaction
```

## Examples
```
/recurring create Monthly rent $3,500 to Landlord Corp on the 1st
/recurring create Quarterly insurance $1,200 to State Farm every 3 months starting 4/1
/recurring pause Monthly rent
/recurring list
```

## Behavior

Use the **Accountant** agent with the **Bookkeeping** skill.

### List Mode (default)
1. Fetch all recurring transactions: `qbRecurringTransaction(action="read")`
2. Present in a table:

| Name | Type | Amount | Frequency | Next Date | Status |
|------|------|--------|-----------|-----------|--------|

3. Flag any that are paused or have errors

### Create Mode
1. Parse the description for: transaction type, amount, vendor/customer, frequency, start date
2. Look up vendor/customer: `qbMasterData`
3. Check agent memory for the correct expense/income account: `agentMemory`
4. Confirm with user: "Create recurring [Type] for $X to [Vendor] every [Frequency] starting [Date]?"
5. Create: `qbRecurringTransaction(action="create")` with:
   - `recurType`: "Automated" (posts automatically) or "Reminder" (sends notification)
   - `interval`: frequency (Monthly, Weekly, Quarterly, Yearly)
   - `startDate`, `endDate` (optional)
   - Full transaction details (vendor, account, amount, lines)
6. Update agent memory with the recurring pattern

### Pause / Resume / Delete Mode
1. Search for the recurring transaction by name: `qbRecurringTransaction(action="read")`
2. Match by name (fuzzy match if needed)
3. Confirm the action with user
4. Execute: `qbRecurringTransaction(action="update")` to toggle active/inactive, or `action="delete"` to remove

## Safety
- Creating automated recurring transactions requires explicit user confirmation
- Recommend "Reminder" type for new setups so the user can verify the first few cycles
- Deleting a recurring transaction does NOT void past transactions it created
- Always show the matched transaction before pausing/deleting to confirm it's the right one
