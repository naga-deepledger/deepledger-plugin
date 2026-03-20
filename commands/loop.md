---
name: loop
description: Run the autonomous accounting loop — processes all pending tasks, records transactions, escalates unknowns to humans, and self-improves
agent: accountant
---

# Autonomous Accounting Loop

You are running the autonomous accounting cycle. Execute all phases systematically.

## Execution Steps

### 1. Gather pending work
- Fetch AI tasks: `fetchAiTasks(status: "scheduled")` and `fetchAiTasks(status: "queued")`
- Check for human replies: `contactHuman(action: "check")`
- Fetch new documents: `fetchDocuments()`
- **Fetch approved reviews**: `fetchApprovedReviews({ action: "list" })` — items CPA approved in portal

### 2. Process human replies first
For each answered question:
- Read the reply
- Resume the blocked task with the new information
- Update agent memory with the learned pattern

### 2b. Record CPA-approved transactions (HIGHEST PRIORITY)
For each item returned by `fetchApprovedReviews`:
- Use `effectiveCategory` (CPA's approved category) as the QB account name
- Load QB master data to get the account ID for that category name
- Check for duplicates via `qbFetchTransactions` using the description + date
- If no duplicate: record in QB using the appropriate tool:
  - `type: "expense"` → `qbExpense`
  - `type: "income"` → `qbSalesReceipt` or `qbDeposit`
- On success: call `fetchApprovedReviews(action: "mark_recorded", reviewItemId, qbTransactionId)`
- Update `agentMemory` with the confirmed vendor→category mapping (high confidence)
- Log the action via `agentLog`

### 3. Process each task by priority (urgent > high > normal > low)
For each task:
- Load client memory: `agentMemory(operation: "read")`
- Load QB master data: `qbMasterData(entityTypes: ["account", "vendor", "customer", "item"])`
- Determine task type and execute:
  - **Record transaction**: Parse → infer type → check duplicates → compute confidence score → record
  - **Run report**: Pull QB reports → analyze → summarize
  - **Process document**: Read via signed URL → extract data → match vendor → record → attach
  - **Transfer funds**: Use qbTransfer (NOT JournalEntry)
  - **Create estimate/quote**: Use qbEstimate
  - **Create purchase order**: Use qbPurchaseOrder
  - **Manage recurring**: Use qbRecurringTransaction (list/create/delete via API)
- Confidence >= 80%: execute autonomously and log
- Confidence 60-79%: execute but flag for review queue
- Confidence < 60%: `contactHuman(action: "send")` with specific question, then move on

### 4. Learn from this cycle
- Update agentMemory with any new vendor→category mappings
- Process review queue feedback (approvals = reinforce, rejections = correct)
- After 3 consistent categorizations for a vendor, mark as confident

### 5. Report
Summarize what happened this cycle:
- Tasks processed / completed / escalated / failed
- Confidence scores for each action taken
- Run `/health-check` if significant financial changes were made

### 6. Self-assess
- Were there repeated escalations of the same type? Note the pattern
- Were there any errors? Log them for debugging
- What could be automated better next cycle?
