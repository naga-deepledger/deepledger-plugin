---
name: loop
description: Run the autonomous accounting loop — processes all pending tasks, records transactions, escalates unknowns to humans, and self-improves
agent: accountant
---

# Autonomous Accounting Loop

You are running the autonomous accounting cycle. Execute all phases systematically.

## Execution Steps

### 0. Load global accounting rules (FIRST — do before anything else)
Read client-specific memory for the active organization:
```
agentMemory(operation: "read", category: "categorization")
agentMemory(operation: "read", category: "vendor")
agentMemory(operation: "read", category: "process")
```

**Global firm-wide categorization rules (always apply regardless of client):**

Account Mapping Rules:
- AWS / GCP / Azure / Vercel / Render / Heroku → **Cloud Hosting/Software expense** (NOT Utilities)
- FedEx / UPS / USPS → **Shipping & Delivery**
- DoorDash / Uber Eats → **Meals**
- Uber / Lyft → **Travel**
- Office Depot / Staples / Amazon (office supplies) → **Office Supplies expense**
- Zoom / Slack / Google Workspace / Microsoft 365 → **Software/SaaS expense**
- WeWork / Regus / office lease → **Rent expense**

Tool Usage Rules:
- "bill from vendor" → `qbBill` (AP). "invoice to customer" → `qbInvoice` (AR)
- "got paid / received payment" → `qbReceivePayment` (fetch open invoices first)
- "paid / bought / spent" → `qbExpense` (direct) or check for open bill → `qbBillPayment`
- "report / how much / spending" → `qbReports` (P&L for spending, Balance Sheet for position)
- Ambiguous "bill" with unknown entity → flag for review in review_queue to clarify customer vs vendor

Workflow Safety Rules:
- **ALWAYS** verify source document exists before recording (documents table, bank feed, or sufficient user detail)
- **ALWAYS** run `qbFetchTransactions` before creating any transaction (duplicate check)
- **ALWAYS** verify vendor/customer exists via `qbMasterData` before referencing in transaction
- **ALWAYS** attach source document to QB transaction after recording via `qbGetUploadUrl`
- Confidence ≥ 80% + document-backed: execute autonomously. 60-79%: execute + flag for review. < 60% or no document: skip and move on to next task

### 1. Gather pending work
- Fetch AI tasks: `fetchWorkQueue(source: "tasks", status: "scheduled")`
- Fetch new documents: `fetchDocuments()`
- **Fetch approved reviews**: `fetchWorkQueue(source: "approvedReviews")` — items CPA approved in portal

### 2. Record CPA-approved transactions (HIGHEST PRIORITY)
For each item returned by `fetchApprovedReviews`:
- Use `effectiveCategory` (CPA's approved category) as the QB account name
- Load QB master data to get the account ID for that category name
- Check for duplicates via `qbFetchTransactions` using the description + date
- If no duplicate: record in QB using the appropriate tool:
  - `type: "expense"` → `qbExpense`
  - `type: "income"` → `qbSalesReceipt` or `qbDeposit`
- On success:
  1. Call `fetchApprovedReviews(action: "mark_recorded", reviewItemId, qbTransactionId)`
- Update `agentMemory` with the confirmed vendor→category mapping (high confidence)

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
- Confidence >= 80%: execute autonomously
- Confidence 60-79%: execute but flag for review queue
- Confidence < 60%: skip and move on to next task

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
- Were there any errors? Note them for debugging
- What could be automated better next cycle?
