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
- Call `contactHuman` with `message_type: "clarification"` explaining what you know and what you need
- Group multiple low-confidence transactions into one message if possible (batch questions)
- Wait for reply before recording

### 4. Update Memory

After each successfully recorded transaction, call `agentMemory` (operation: write) to reinforce the pattern:
- category: "categorization"
- subject: vendor name
- memory: "Vendor X → Account Y (expense type Z) — recorded from bank feed"
- confidence: based on match strength

### 5. Log & Notify

Always call `agentLog` with a summary:
- action: "Bank feed triage — X recorded, Y flagged, Z need human input"
- outcome: counts breakdown
- logLevel: "success" if recorded > 0, "warn" if most needed human input

If significant work was done (any transactions recorded), call `notifyUser`:
- type: "success"
- title: "Bank Feed Processed"
- body: "Processed N transactions: X auto-recorded to QuickBooks, Y flagged for review, Z need your input"
- actionLabel: "View Review Queue" (if Y > 0) or "View Bank Feed"
- actionUrl: "/review" or "/bank-feed"

## Safety Rules

- ALWAYS check for duplicates before recording (qbFetchTransactions with date + amount + vendor)
- NEVER record a transaction if qbMasterData lookup fails
- If QB is not connected, use `contactHuman` to notify CPA and stop

## Usage

```
/bank-feed
/bank-feed Acme Corp
/bank-feed --org <org_id>
```
