---
name: bank-feed
description: Triage bank feed transactions for an organization using AI categorization — records high-confidence transactions to QB, flags medium-confidence for review, asks CPA for low-confidence ones
---

# /bank-feed — Bank Feed Triage

Runs AI-assisted categorization of unprocessed bank feed transactions. Reads from `teller_transactions`, checks `agent_memory` for vendor patterns, then records, flags, or escalates each transaction.

## Steps

### 1. Identify Organization

If no org is specified, ask: "Which client would you like to run bank feed triage for?" then use `qbMasterData` to confirm QB connection.

### 2. Run Categorization

Call `qbCategorizeBankFeed` with the organization ID. This returns per-transaction recommendations:
- `record` — confidence ≥ 80%, safe to auto-record
- `flag_for_review` — confidence 60–79%, CPA should approve before recording
- `human_categorize` — confidence < 60%, agent needs more context

### 3. Process Each Result

For **`record`** actions:
- Call `qbExpense` (for debits) or `qbSalesReceipt` (for credits) using `suggested_category` and `suggested_account`
- Require qbMasterData lookup + duplicate check before recording (write safety)
- Update agent_memory to reinforce the vendor→category pattern

For **`flag_for_review`** actions:
- Call `qbFlagForReview` with `confidence_score`, `suggested_category`, and `reason`
- These appear in the portal Review Queue for CPA approval

For **`human_categorize`** actions:
- Call `qbFlagForReview` explaining what you know and what you need
- Group multiple low-confidence transactions into one review_queue entry if possible
- These will be reviewed in the next portal cycle

### 4. Update Memory

After each successfully recorded transaction, call `agentMemory` (operation: write) to reinforce the pattern:
- category: "categorization"
- subject: vendor name
- memory: "Vendor X → Account Y (expense type Z) — recorded from bank feed"
- confidence: based on match strength

## Safety Rules

- ALWAYS check for duplicates before recording (qbFetchTransactions with date + amount + vendor)
- NEVER record a transaction if qbMasterData lookup fails
- If QB is not connected, flag for review and stop

## Usage

```
/bank-feed
/bank-feed Acme Corp
/bank-feed --org <org_id>
```
