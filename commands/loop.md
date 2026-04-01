# /loop

Run the autonomous bookkeeping loop — checks for pending work, processes transactions, flags uncertainties, and learns from results.

## Usage
```
/loop                          # Run one full cycle
/loop status                   # Check worklog status without processing
/loop reset                    # Reset worklog and start fresh
```

## Behavior

Use the **Accountant** agent with the **Bookkeeping** skill.
Follow the `getGuide(guideType="autonomous_loop")` workflow.

### Step 0: BOOTSTRAP CHECK
- `agentMemory(operation="read", type="bootstrap_status")`
- If NOT bootstrapped:
  - Warn: "This client has not been bootstrapped. The agent has no learned mappings — most transactions will be flagged for review."
  - Recommend: "Run `/bootstrap` first for much better accuracy."
  - If user confirms to proceed anyway, continue (but expect high flag rate)
  - If running unattended (scheduled), auto-run bootstrap before first cycle

### Step 1: INITIALIZE — Resume or Start
- Read worklog: `agentMemory(operation="read", type="worklog")`
- If a previous cycle has `status="running"`:
  - Check `lastCompletedStep` to determine where it crashed
  - Resume from the NEXT step after `lastCompletedStep`
  - Increment `crashRecoveryCount` in worklog
  - If `crashRecoveryCount` >= 3 for the same step, flag the step as broken and skip it
- If no worklog or `status="completed"`, start a fresh cycle
- Write/update worklog with `status="running"`, increment `cycleCount`
- Record `startedAt` timestamp for this cycle

### Step 2: CHECK — Fetch Pending Work
Gather work from three sources (in priority order):

1. **CPA-approved reviews** (highest priority, safest):
   `fetchWorkQueue(source="approvedReviews")`

2. **Scheduled AI tasks**:
   `fetchWorkQueue(source="tasks")`

3. **Unprocessed bank feed**:
   `bankFeed(action="fetch")`

4. **Changed QB data** (detect external changes):
   `qbChangeDataCapture` with `lastProcessedTimestamp` from worklog

If zero pending items across all sources, update worklog to `status="completed"` and exit.

After completing this step, update worklog: `lastCompletedStep="check"`

### Step 3: ANALYZE — Read Context
For each pending item:
- `agentMemory` — check vendor/customer account mappings and confidence (upvote count)
- `agentMemory` — check client preferences and special rules
- `fetchDocuments` — read linked documents if available
- Use enrichment data from bank feed (already includes memory matches)

Confidence levels:
- **High** (5+ upvotes): auto-categorize
- **Medium** (3-4 upvotes): auto-categorize with note
- **Low** (1-2 upvotes): proceed with caution, consider flagging
- **None** (0 or new vendor): flag for CPA review

After completing this step, update worklog: `lastCompletedStep="analyze"`

### Step 4: ACT — Record Transactions

**CPA-approved items** (process first):
1. Record using the `effectiveCategory` from the approval
2. Mark as recorded: `fetchWorkQueue(source="markRecorded", reviewItemNumber=N, qbTransactionId=ID)`

**High-confidence bank feed items**:
1. `qbMasterData` — lookup vendor/account IDs
2. `qbFetchTransactions` — duplicate check
3. Record with the top-voted account mapping
4. Mark as recorded

**Always** follow write safety: lookup → duplicate check → record.
Use `qbBatch` when recording 3+ similar transactions.

**Error handling for individual transactions:**
- If a single transaction fails, log the error in worklog `failedItems[]` and continue with the next
- Do NOT abort the entire cycle for one failed transaction
- Failed items will be retried in the next cycle (max 3 retries per item before flagging for CPA)

After completing this step, update worklog: `lastCompletedStep="act"`, record `itemsRecorded` count

### Step 5: ESCALATE — Flag Uncertain Items
For low-confidence or unknown items:
- `bankFeed(action="flag")` with specific `aiReasoning`:
  - "New vendor not in memory"
  - "Amount $X is 3x the usual $Y for this vendor"
  - "Multiple possible categories, no clear winner"
  - "Description is ambiguous"

Include `suggestedCategory` when you have a reasonable guess.

After completing this step, update worklog: `lastCompletedStep="escalate"`, record `itemsFlagged` count

### Step 6: LEARN — Update Memory
After processing:
- **Upvote** account mappings that were used successfully
- **Write** new vendor memories for first-time vendors
- **Update** vendor memories when CPA corrects a category
- **Store** new client preferences discovered

After completing this step, update worklog: `lastCompletedStep="learn"`

### Step 7: AUDIT & REPORT — Health Check
1. `qbReconciliationCheck` on all active bank/CC accounts
2. Update worklog: `status="completed"`, final counts, timestamp
3. Compare health scores to previous cycle
4. Present cycle summary:
   - Transactions recorded: X
   - Items flagged for review: Y
   - Health scores (with delta from last cycle)
   - Anomalies detected
   - Memory entries updated

## Status Mode
When called with `status`:
- Read the worklog memory entry
- Show: last cycle time, items processed, current status, health scores
- Do NOT process any transactions

## Reset Mode
When called with `reset`:
- Update worklog to `status="idle"`, reset counters
- Confirm with user before resetting

## Worklog Schema

The worklog memory entry tracks cycle state for crash recovery:

```
{
  status: "running" | "completed" | "idle",
  cycleCount: number,
  lastCompletedStep: "check" | "analyze" | "act" | "escalate" | "learn" | "audit",
  startedAt: ISO timestamp,
  completedAt: ISO timestamp,
  lastProcessedTimestamp: ISO timestamp (for qbChangeDataCapture),
  itemsRecorded: number,
  itemsFlagged: number,
  failedItems: [{ id, reason, retryCount }],
  crashRecoveryCount: number,
  healthScores: { accountName: score }
}
```

## Safety
- Process CPA-approved items FIRST — they are verified and safe
- NEVER override a CPA-approved category with agent judgment
- NEVER skip the duplicate check
- Flag when in doubt — a false escalation costs 30 seconds, a wrong recording costs hours
- Update worklog after EVERY step — this is how crash recovery works
- If a step fails 3 times consecutively, skip it and flag for CPA review
- Individual transaction failures do NOT abort the cycle — log and continue
