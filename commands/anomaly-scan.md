---
description: Detect suspicious or unusual transactions using historical patterns from agent memory
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [days|period]
---

Run an anomaly detection scan using the Anomaly Detection skill.

If $ARGUMENTS specifies a number of days (e.g., "30", "7", "90"), use that as the look-back period.
If $ARGUMENTS specifies a date range (e.g., "Jan 2026"), use that period.
Default: last 30 days.

Execute autonomously — no intermediate questions. Load memory, scan transactions, flag anomalies, generate report.

Steps:
1. Load agent memory for this client (vendor patterns, typical amounts, frequency)
2. Calculate date range: today minus look-back period
3. Fetch recent transactions:
   - qbFetchTransactions Purchase (expenses/checks) for the period
   - qbFetchTransactions Bill for the period
   - qbFetchTransactions JournalEntry for the period (manual adjustments)
4. Apply ALL anomaly detectors from the Anomaly Detection skill:
   - Duplicate detection (same vendor + amount + date/near-date)
   - Round number alerts (>$1,000 with suspiciously round totals)
   - New vendor with large amount (>$1,000 and no memory baseline)
   - Weekend/holiday transactions
   - Amount spikes (>2x normal for vendor)
   - Category mismatches from memory
   - Large manual journal entries (>$5,000, no document)
   - Split transaction patterns (multiple charges just below a threshold)
5. Score each anomaly and prioritize (Critical/High/Medium/Low)
6. For CRITICAL anomalies: flag in review_queue with high priority
7. For HIGH/MEDIUM: add to review_queue
8. Update agentMemory with new vendor baselines discovered
9. Output the full Anomaly Detection Report

Present findings clearly with: transaction description, amount, date, vendor, reason flagged, and what normal looks like for comparison.
