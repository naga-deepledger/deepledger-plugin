# /bootstrap

Learn a client's accounting patterns from their existing QuickBooks history. Run this once when a new client connects QuickBooks — it seeds the agent memory so the first real bookkeeping run is already accurate.

## Usage
```
/bootstrap                     # Full bootstrap from last 12 months
/bootstrap 6                   # Bootstrap from last 6 months
/bootstrap review              # Show what was learned without activating
/bootstrap status              # Check if this client has been bootstrapped
```

## Why This Exists

Without bootstrap, the agent starts with zero memory — every vendor is unknown, every transaction gets flagged, the CPA is overwhelmed with hundreds of review items on day one.

With bootstrap, the agent reads what the CPA already recorded in QuickBooks and learns from it. First real run: only genuinely new vendors get flagged.

## Behavior

Use the **Accountant** agent with the **Bookkeeping** skill.

### Step 1: CHECK — Is This Client Already Bootstrapped?
- `agentMemory(operation="read", type="bootstrap_status")`
- If `bootstrapped=true` and not forced, warn: "This client was bootstrapped on [date]. Run `/bootstrap reset` to re-learn, or `/bootstrap review` to see current mappings."
- If not bootstrapped, proceed.

### Step 2: EXTRACT — Pull Historical Transactions
Fetch ALL transactions from the requested period (default: 12 months):

```
qbFetchTransactions(type="Expense", startDate="12 months ago", endDate="today", limit=500)
qbFetchTransactions(type="Bill", startDate="12 months ago", endDate="today", limit=500)
qbFetchTransactions(type="BillPayment", startDate="12 months ago", endDate="today", limit=500)
qbFetchTransactions(type="Invoice", startDate="12 months ago", endDate="today", limit=500)
qbFetchTransactions(type="SalesReceipt", startDate="12 months ago", endDate="today", limit=500)
qbFetchTransactions(type="ReceivePayment", startDate="12 months ago", endDate="today", limit=500)
qbFetchTransactions(type="Deposit", startDate="12 months ago", endDate="today", limit=500)
qbFetchTransactions(type="JournalEntry", startDate="12 months ago", endDate="today", limit=500)
qbFetchTransactions(type="Transfer", startDate="12 months ago", endDate="today", limit=500)
```

Paginate through all results if a type has more than 500 records.

### Step 3: ANALYZE — Extract Patterns

For each transaction, extract and aggregate:

**Vendor Patterns (from Expenses, Bills, BillPayments):**
```
{
  vendorName: "Office Depot",
  vendorId: "123",
  accountMapping: "Office Supplies",
  accountId: "456",
  transactionType: "Expense",          // how the CPA records this vendor
  frequency: 23,                        // times seen in the period
  amountRange: { min: 45.00, max: 892.00, avg: 187.50 },
  paymentMethod: "Visa *4521",          // which card/bank
  lastSeen: "2026-03-15"
}
```

**Customer Patterns (from Invoices, SalesReceipts, ReceivePayments):**
```
{
  customerName: "Client ABC",
  customerId: "789",
  incomeAccount: "Consulting Income",
  accountId: "101",
  transactionType: "Invoice",           // Invoice vs SalesReceipt
  frequency: 12,
  amountRange: { min: 2500.00, max: 5000.00, avg: 3750.00 },
  lastSeen: "2026-03-20"
}
```

**Recurring Patterns (from JournalEntries):**
```
{
  description: "Monthly depreciation",
  accounts: ["Depreciation Expense", "Accumulated Depreciation"],
  amount: 1250.00,
  frequency: "monthly",
  lastSeen: "2026-03-31"
}
```

**Transfer Routes (from Transfers):**
```
{
  fromAccount: "Business Checking",
  toAccount: "Payroll Account",
  frequency: 4,
  typicalAmount: 15000.00
}
```

### Step 4: PRESENT — Show Summary to CPA for Review

Display the learned patterns in a table for CPA confirmation:

```
Bootstrap Summary — [Client Name]
══════════════════════════════════
Analyzed: 847 transactions over 12 months
Vendors learned: 45
Customers learned: 12
Recurring patterns: 6
Transfer routes: 3

TOP VENDOR MAPPINGS (by frequency):
┌──────────────────┬─────────────────┬──────┬──────────┬──────────┐
│ Vendor           │ Account         │ Type │ Freq     │ Avg Amt  │
├──────────────────┼─────────────────┼──────┼──────────┼──────────┤
│ Office Depot     │ Office Supplies │ Exp  │ 23x      │ $187.50  │
│ AWS              │ Cloud Services  │ Exp  │ 12x      │ $2,340   │
│ Uber             │ Travel          │ Exp  │ 15x      │ $34.50   │
│ Landlord Corp    │ Rent            │ Bill │ 12x      │ $3,500   │
│ Random LLC       │ Miscellaneous   │ Exp  │ 1x       │ $450.00  │
└──────────────────┴─────────────────┴──────┴──────────┴──────────┘

⚠  LOW-FREQUENCY VENDORS (seen only 1-2 times — may be one-offs):
   - Random LLC → Miscellaneous (1x)
   - Temp Agency → Contract Labor (2x)

Any corrections before I activate these mappings?
```

**Wait for CPA confirmation.** This is the "show a summary" part — let the CPA catch any miscategorizations before they become learned behavior.

### Step 5: SEED — Write to Agent Memory

After CPA confirms (or corrects):

For each vendor mapping:
```
agentMemory(operation="write", type="vendor", data={
  vendorName: "Office Depot",
  vendorId: "123",
  accountName: "Office Supplies",
  accountId: "456",
  transactionType: "Expense",
  upvotes: min(frequency, 5),            // CAP AT 5 — earn higher trust through usage
  amountRange: { min: 45.00, max: 892.00, avg: 187.50 },
  source: "bootstrap",
  bootstrapDate: "2026-04-01"
})
```

For each customer mapping:
```
agentMemory(operation="write", type="customer", data={
  customerName: "Client ABC",
  customerId: "789",
  incomeAccount: "Consulting Income",
  accountId: "101",
  transactionType: "Invoice",
  upvotes: min(frequency, 5),             // CAP AT 5
  amountRange: { min: 2500.00, max: 5000.00, avg: 3750.00 },
  source: "bootstrap",
  bootstrapDate: "2026-04-01"
})
```

For each recurring pattern:
```
agentMemory(operation="write", type="client", data={
  pattern: "recurring_journal_entry",
  description: "Monthly depreciation",
  accounts: [...],
  amount: 1250.00,
  frequency: "monthly",
  source: "bootstrap"
})
```

**Upvote cap: 5.** Even if a vendor appeared 50 times, the bootstrap caps at 5. The agent must earn higher confidence through real-time usage. This protects against historical miscategorizations — a CPA correction in the review queue can still override a bootstrap mapping.

### Step 6: MARK — Record Bootstrap Completion

```
agentMemory(operation="write", type="bootstrap_status", data={
  bootstrapped: true,
  date: "2026-04-01",
  period: "12 months",
  transactionsAnalyzed: 847,
  vendorsLearned: 45,
  customersLearned: 12,
  recurringPatterns: 6,
  transferRoutes: 3,
  cpAReviewed: true
})
```

### Step 7: REPORT — Activation Summary

```
Bootstrap Complete — [Client Name]
═══════════════════════════════════
Memory seeded from 847 transactions (12 months)

  ✓ 45 vendor → account mappings (capped at 5 upvotes each)
  ✓ 12 customer → income account mappings
  ✓ 6 recurring patterns stored
  ✓ 3 transfer routes learned
  ✓ Amount ranges set for anomaly detection

Ready for /bank-feed or /loop — the agent will:
  • Auto-categorize known vendors (5 upvotes = medium-high confidence)
  • Flag unknown vendors for CPA review
  • Detect anomalous amounts (3x outside learned range)
  • Earn higher confidence through successful recordings

Next recommended step: /bank-feed or /loop
```

## Review Mode

When called with `review`:
- Read all bootstrap-sourced memories: `agentMemory(operation="read", type="vendor", filter="source:bootstrap")`
- Show current mappings table with current upvote counts
- Allow CPA to correct any mapping: update the memory entry
- Do NOT re-fetch from QB or overwrite non-bootstrap memories

## Status Mode

When called with `status`:
- Read bootstrap status: `agentMemory(operation="read", type="bootstrap_status")`
- Show: date, transactions analyzed, mappings count, whether CPA reviewed
- Show current confidence distribution:
  ```
  High confidence (5+ upvotes):   32 vendors  (71%)
  Medium confidence (3-4):         8 vendors  (18%)
  Low confidence (1-2):            5 vendors  (11%)
  ```

## Reset Mode

When called with `reset`:
- Confirm with CPA: "This will delete all bootstrap-sourced memories and re-learn from scratch. Continue?"
- Delete all memories with `source: "bootstrap"` that haven't been upvoted beyond the bootstrap cap
- Preserve memories that were upvoted through real usage (upvotes > 5)
- Re-run the full bootstrap

## Safety
- NEVER activate bootstrap mappings without CPA review — always show the summary first
- Cap upvotes at 5 — the agent must earn trust through real-time accuracy
- Preserve amount ranges for anomaly detection — flag if a new transaction is 3x outside the learned range
- Low-frequency vendors (1-2 occurrences) get a warning flag — they may be one-offs, not patterns
- CPA corrections in the portal ALWAYS override bootstrap mappings
- Bootstrap memories are tagged with `source: "bootstrap"` so they can be distinguished from real-time learning
