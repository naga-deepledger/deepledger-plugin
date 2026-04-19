---
name: client-onboarding
description: Bootstrap a new client by analyzing historical QuickBooks data to seed the AI agent's memory. Use when the user mentions bootstrapping, setting up a new client, or learning from history.
---

# Client Onboarding Skill

Extract historical transactions to learn vendor mappings, customer income accounts, and recurring patterns. Seed the agent's memory so it operates accurately from day one.

## Trigger

Activate this skill when the user wants to:
- Bootstrap a new client
- Set up a newly connected QuickBooks account
- Learn from existing QuickBooks history
- Re-run the onboarding analysis

## Workflow: Client Bootstrap (First-Time Setup)

Run once when a new client connects QuickBooks. Seeds agent memory from historical transactions so the agent is accurate from day one.

### Pre-check
- `agentMemory(operation="read", type="bootstrap_status")` — skip if already bootstrapped, unless user specifically asks to re-run.

### Extract
Pull all transactions from the last 12 months (configurable) using `qbFetchTransactions` for each type: Expense, Bill, BillPayment, Invoice, SalesReceipt, ReceivePayment, Deposit, JournalEntry, Transfer.

### Analyze
For each transaction, extract:
- **Vendor mappings**: vendor → expense account, transaction type, frequency, amount range
- **Customer mappings**: customer → income account, transaction type, frequency, amount range
- **Recurring patterns**: repeated JEs (depreciation, accruals) with consistent amounts
- **Transfer routes**: common account-to-account transfer paths

### Present to CPA
Show a summary table of all learned mappings. Flag low-frequency vendors (1-2 occurrences) as potential one-offs. **Wait for CPA confirmation before activating.**

### Seed Memory
Write each mapping to `agentMemory` with:
- `upvotes`: capped at **5** regardless of historical frequency — the agent must earn higher trust through real-time usage
- `source: "bootstrap"` — tags the memory as bootstrap-derived so it can be distinguished from real-time learning
- `amountRange`: min/max/avg from history — used for anomaly detection (flag if new transaction is 3x outside range)

### Mark Complete
Write `bootstrap_status` to memory with date, counts, and `cpaReviewed: true`.

## Safety Checklist
- [ ] Verify bootstrap hasn't already been run (prevent duplicate seeding)
- [ ] Present mappings to CPA for approval before committing to agent memory
- [ ] Cap bootstrap upvotes at 5
