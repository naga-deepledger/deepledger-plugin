---
description: Weekly AI accuracy self-audit — measures categorization accuracy, identifies error patterns, and updates learning priorities
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period — default last 7 days]
---

Run a self-audit of AI categorization accuracy. Measures how well the AI
has been categorizing transactions by comparing AI suggestions against
human-approved/rejected outcomes.

If $ARGUMENTS contains a period, use it. Default to the last 7 days.

Execute ALL steps autonomously — pull all data, analyze, and present a
complete accuracy report in one response.

## Steps

**Phase 1: Pull Review Queue Outcomes**

1. Use qbFetchTransactions or the review queue data (via agentMemory or
   direct query) to find all transactions that went through the review
   queue in the audit period
2. Categorize outcomes:
   - **Accepted as-is**: AI category was approved without changes
   - **Corrected**: AI category was approved but human changed the category
   - **Rejected**: AI suggestion was rejected entirely
   - **Pending**: Still awaiting human review (exclude from accuracy calc)

**Phase 2: Calculate Accuracy Metrics**

Compute the following:

| Metric | Formula |
|--------|---------|
| **Overall Accuracy** | (Accepted as-is) / (Accepted + Corrected + Rejected) |
| **Acceptance Rate** | (Accepted + Corrected) / Total reviewed |
| **Correction Rate** | Corrected / Total reviewed |
| **Rejection Rate** | Rejected / Total reviewed |
| **Confidence Calibration** | Average confidence on correct vs incorrect predictions |

Also compute per-category accuracy:
- For each expense/income category, what % of AI suggestions were correct?
- Identify categories where AI consistently gets it wrong

And per-vendor accuracy:
- Which vendors does AI categorize correctly every time?
- Which vendors does AI struggle with? (>1 correction in audit period)

**Phase 3: Error Pattern Analysis**

For every correction or rejection, analyze the pattern:

1. **Systematic errors**: Same mistake repeated (e.g., always putting
   "AWS" in "Office Supplies" instead of "Cloud Services")
   → These are the highest-priority fixes — update agentMemory immediately
2. **Novel situations**: First-time vendor or unusual transaction type
   → Expected errors, not a learning failure
3. **Judgment calls**: Category was reasonable but human preferred different
   → Note the preference, update memory with human's choice
4. **Confidence mismatches**: High-confidence predictions that were wrong
   → Most concerning — indicates flawed pattern matching

**Phase 4: Self-Correct**

For each systematic error found:
1. Read current agentMemory for the vendor/category pattern
2. Update with the corrected pattern:
   ```
   agentMemory(
     operation: "update",
     memoryId: <existing_id>,
     memory: "<corrected pattern JSON>",
     confidence: 90
   )
   ```
   If no memory exists, create one:
   ```
   agentMemory(
     operation: "write",
     category: "categorization",
     subject: "<vendor_name>",
     memory: "<JSON: vendor→correct_category mapping based on human corrections>",
     confidence: 85
   )
   ```
3. Record each self-correction in agentMemory

**Phase 5: Store Audit Results**

Save the audit results to agentMemory for the dashboard to display:

```
agentMemory(
  operation: "write",
  category: "process",
  subject: "self_audit:last_result",
  memory: "<JSON string: {
    \"accuracy_rate\": <0.0-1.0>,
    \"total_reviewed\": <N>,
    \"accepted\": <N>,
    \"corrected\": <N>,
    \"rejected\": <N>,
    \"week_ending\": \"<YYYY-MM-DD>\",
    \"systematic_errors_fixed\": <N>,
    \"top_error_categories\": [\"cat1\", \"cat2\"],
    \"top_error_vendors\": [\"vendor1\", \"vendor2\"],
    \"confidence_calibration\": {\"correct_avg\": <N>, \"incorrect_avg\": <N>}
  }>",
  confidence: 100
)
```

If a memory with category="process" and subject="self_audit:last_result"
already exists, use operation: "update" with the memoryId.

Also save a dated entry for sparkline history:

```
agentMemory(
  operation: "write",
  category: "self_audit",
  subject: "<YYYY-MM-DD of audit>",
  memory: "<JSON string: {\"accuracy_rate\": <0.0-1.0>, \"total_reviewed\": <N>}>",
  confidence: 100
)
```

**Phase 6: Present Report**

```
## AI Self-Audit Report — [Date Range]
**Org:** [Client Name]

### Accuracy Summary
Overall Accuracy: XX% (N/M transactions correct)
Acceptance Rate:  XX% | Correction Rate: XX% | Rejection Rate: XX%

### Confidence Calibration
Avg confidence on CORRECT predictions:   XX%
Avg confidence on INCORRECT predictions: XX%
Gap: XX points (larger gap = well-calibrated; small gap = overconfident)

### Error Breakdown
Systematic errors (fixable): N — [details]
Novel situations (expected):  N — [details]
Judgment calls (preference):  N — [details]

### Category Accuracy
| Category | Correct | Incorrect | Accuracy |
|----------|---------|-----------|----------|
| [cat]    | N       | N         | XX%      |

### Vendor Accuracy (problem vendors only)
| Vendor | Times Wrong | Correct Category | AI Guessed |
|--------|-------------|-----------------|------------|
| [name] | N           | [correct]       | [wrong]    |

### Self-Corrections Applied
- Updated [vendor] pattern: [old] → [new]
- Created new rule: [vendor] → [category]

### Trend
Current accuracy: XX% | Prior audit: XX% | Direction: ↑/↓/→

### Recommendations
1. [Most impactful improvement — specific action]
2. [Second improvement]
```

## Integration with Autonomous Loop

This self-audit should run:
1. Weekly as part of the autonomous loop
2. Before any month-end close (to ensure accuracy is high before closing)
3. On-demand when the user runs `/self-audit`

The dashboard's "AI Learning Progress" card reads the stored audit results
to display accuracy trends and the sparkline chart.

## Target Benchmarks (Digits/Pilot Comparison)

| Metric | Target | Digits Reference |
|--------|--------|-----------------|
| Overall Accuracy | ≥95% | 96.5% |
| Correction Rate | ≤5% | ~3% |
| Rejection Rate | ≤2% | ~0.5% |
| Confidence Calibration Gap | ≥15 points | N/A |

If accuracy drops below 90% for 2 consecutive audits, recommend a
comprehensive review of agentMemory patterns and consider resetting
low-confidence rules.
