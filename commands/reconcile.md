# /reconcile

Reconcile a bank or credit card account — clean up the account, record missing transactions, then complete the reconciliation in QuickBooks Online.

## Usage
```
/reconcile                        # Health check all bank and credit card accounts
/reconcile <account name>         # Full reconciliation workflow for a specific account
/reconcile health                 # Health check only — no recording or reconciling
/reconcile health <account name>  # Health check a specific account
```

## Behavior

Use the **Accountant** agent with the **Bank Reconciliation** skill.

Reconciliation runs in two phases:
1. **Prep phase** — health check, record missing bank feed transactions, resolve duplicates and uncategorized items (MCP tools)
2. **Reconcile phase** — mark transactions cleared and finalize in the QuickBooks Online browser UI (requires browser access)

---

### Phase 1: Prep

#### Step 1 — Account Health Check
Run `/health-check` (or `/health-check <account name>` for a specific account). Target: all accounts >= 90 before proceeding. Do not open the reconciliation screen if any account is < 80.

#### Step 2 — Process Bank Feed
Run `/bank-feed` to record missing transactions for the period. All unrecorded bank feed items must be recorded before reconciling. See `/bank-feed` for the full confidence-scoring and flagging workflow.

#### Step 3 — Resolve Duplicate Flags
1. `qbFetchTransactions` — pull both entries and confirm they are truly the same (same amount, vendor, purpose)
2. Check whether the transaction type supports voiding. **`qbVoidTransaction` supports**: `BillPayment`, `Invoice`, `Payment`, `SalesReceipt`, `CreditMemo`, `Purchase`, `RefundReceipt`, `Transfer`
   - `Bill`, `JournalEntry`, `Deposit`, `Expense`, `VendorCredit` **cannot be voided via tools** — flag for CPA to handle directly in QB
3. Confirm with the user before voiding
4. `qbVoidTransaction` if supported and confirmed
5. Re-run `qbAccountHealth` to verify the flag is cleared

#### Step 4 — Resolve Uncategorized Flags
1. `qbFetchTransactions` — find entries booked to "Ask My Accountant" or "Uncategorized"
2. `agentMemory` — check for vendor-to-account mapping
3. If confident: re-categorize with the correct account (confirm with user first)
4. If uncertain: `flagForReview` with specific aiReasoning

---

### Phase 2: Reconcile in QuickBooks Online (Browser)

> This phase requires browser access to QuickBooks Online. Claude can guide you through the steps but cannot click or interact with the QB UI directly.

Once the account health score is clean and all bank feed items are recorded:

1. Open QuickBooks Online in your browser → Bookkeeping → Reconcile
2. Select the account to reconcile
3. Enter statement details: beginning balance, ending balance, statement end date
4. Check off each transaction that appears on the bank statement — the running difference should approach $0.00
5. If a difference remains, use `qbFetchTransactions` to investigate suspicious items
6. **Stop and review** — Claude will report the current difference and uncleared items. Do not click "Finish now" yet.
7. **Explicit confirmation required** — Claude will only proceed to finalize after you explicitly confirm the difference is acceptable and instruct completion
8. After finalization, QB generates a reconciliation report PDF — save or attach for audit records

> **Hard rule: Claude will never instruct you to click "Finish now" without your explicit confirmation — even if the difference is $0.00.**

---

### /reconcile health — Health Check Only
Delegates to `/health-check` — no transactions recorded, reconciliation screen not opened. Use `/health-check` directly to assess account status without making changes.

## Safety
- Health check must be run and flags resolved before opening the reconciliation screen
- Duplicate check required before recording any bank feed item
- Confirm voidable transaction type before calling `qbVoidTransaction`
- Never void without confirming it is a true duplicate — two similar charges may both be legitimate
- Never auto-finalize reconciliation — always stop, report, and wait for explicit user instruction
