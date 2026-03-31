# Auto-Categorization Confidence Model

## Overview

This reference defines the confidence scoring model used by DeepLedger's
auto-categorization system. The model combines four evidence sources into
a single confidence score (0-100%) that determines whether to auto-apply
a category, flag for review, or ask the human.

## Evidence Sources (ranked by strength)

### Source 1: Agent Memory — Human-Approved Patterns (strongest)

**How it works:** After a human approves or rejects a categorization in the
Review Queue, the portal writes the decision to `agent_memory` (category:
"categorization", subject: vendor/description). The agent checks this
memory FIRST for every transaction.

**Confidence values:**
- Memory exists + human-approved → **95%** base confidence
- Memory exists + approved 3+ times → **98%** (near-certain)
- Memory exists but was REJECTED → **0%** for that category
  (agent must choose a different category or ask)

**Lookup:** `agentMemory` read with `category="categorization"` and
`subject` matching the vendor name or description.

### Source 2: Transaction History — Consistent Patterns

**How it works:** Fetch the vendor's last 10-20 transactions via
`qbFetchTransactions`. Analyze which QB account was used.

**Confidence values:**
- Same account used 90%+ of the time → **90%** confidence
- Same account used 70-89% of the time → **80%** confidence
- Mixed accounts (50-69% dominant) → **65%** confidence
- No dominant account (<50%) → **40%** confidence (flag for review)

**Key rule:** History confidence is capped at 90% because only
human-approved memory can reach 95%+.

### Source 3: Vendor Name — Single-Purpose Vendors

**How it works:** For new vendors with no history, check if the vendor is
a "single-purpose" vendor (sells only one type of thing).

**Confidence values:**
- Known single-purpose vendor (from categorization.md list) → **75%**
- Vendor name contains strong category signal
  (e.g., name includes "Software", "Insurance", "Supplies") → **70%**
- Multi-purpose vendor (Amazon, Costco, etc.) → **30%** (must use description)

### Source 4: Description Keywords — Weakest Signal

**How it works:** Parse the transaction description for category-indicating
keywords (see categorization.md keyword table).

**Confidence values:**
- Strong keyword match (e.g., "SaaS subscription" → Software) → **60%**
- Moderate keyword match (e.g., "monthly" + tech vendor → Software) → **50%**
- Weak/ambiguous keywords → **35%**
- No keyword signal → **25%**

## Combining Evidence Sources

When multiple sources are available, use the **highest confidence** source,
not an average. The hierarchy is strict:

```
Memory (95-98%) > History (40-90%) > Vendor Name (30-75%) > Keywords (25-60%)
```

**Exception — Negative memory override:** If agent memory says a category
was REJECTED, that overrides ALL other sources for that specific category.
The agent must choose a different category using the remaining sources.

## Decision Thresholds

| Confidence | Action |
|-----------|--------|
| ≥ 90% | **Auto-apply** — Use this category in the proposal without asking. Mark as "(auto-categorized)" |
| 70-89% | **Suggest with flag** — Use this category but flag for Review Queue. Include AI reasoning. |
| 50-69% | **Flag for review** — Suggest the category but flag for human review. Show alternatives. |
| < 50% | **Ask or escalate** — Either ask the user directly or flag with "needs human input" |

## The Learning Loop

The confidence model improves over time through this feedback cycle:

```
1. Agent categorizes transaction
   ↓
2. If confidence < 90%, flags to Review Queue
   ↓
3. Human approves or rejects in portal
   ↓
4. Portal writes decision to agent_memory:
   - Approve: confidence += 5 (up to 95)
   - Reject: confidence -= 15 (down to 10)
   ↓
5. Next time same vendor appears, agent reads memory FIRST
   ↓
6. Approved pattern → 95% confidence → auto-apply (no review needed)
   Rejected pattern → 0% for that category → must choose different
```

**Goal:** Over time, more transactions auto-categorize at 95%+ confidence,
reducing the Review Queue to only genuinely novel or ambiguous items.

## AI Reasoning Format

When flagging a transaction for review, the agent MUST provide detailed
reasoning via the `aiReasoning` field in `qbFlagForReview`. This helps
the CPA make faster decisions.

**Required reasoning structure:**
```
1. Evidence checked: [what sources were consulted]
2. Category chosen: [account name + AcctNum]
3. Why: [specific evidence that led to this choice]
4. Confidence: [X%] from [source name]
5. Alternatives: [if confidence < 80%, list 1-2 other possible categories]
```

**Example reasoning strings:**

High confidence:
```
Memory lookup found "Staples" → "Office Supplies" (AcctNum 6000),
approved by human 4 times. Confidence: 95% from memory. No alternatives
needed.
```

Medium confidence:
```
No memory for "CloudFlare Inc". Transaction history shows 3/4 recent
charges to "Software/SaaS" (AcctNum 6500), 1 to "Web Hosting" (AcctNum
6520). Confidence: 80% from history (dominant pattern). Alternative:
Web Hosting (6520).
```

Low confidence:
```
New vendor "TechServ LLC" — no memory, no transaction history.
Description "consulting services Q1" suggests Professional Services
(AcctNum 6400). Vendor name contains no strong category signal.
Confidence: 55% from description keywords. Alternatives: Software/SaaS
(6500), Repairs & Maintenance (6550).
```

## Accuracy Tracking

The `/self-audit` command measures auto-categorization accuracy by
comparing AI suggestions to human decisions in the Review Queue:

- **Acceptance rate** = approved / (approved + rejected)
- **Correction rate** = approved-with-edit / total-approved
- **Confidence calibration** = does 90% confidence actually mean 90% correct?

Target: 96.5% acceptance rate (matching Digits benchmark).

## Integration Points

This model is used by:
- `/categorize` command — single and bulk categorization
- `/record` command — auto-selects category during transaction recording
- `accountant.md` agent — follows this model for all categorization decisions
- `qbFlagForReview` tool — receives confidence + reasoning for Review Queue
- `/self-audit` command — measures model accuracy over time
- Review Queue portal — displays reasoning and confidence source to CPA
