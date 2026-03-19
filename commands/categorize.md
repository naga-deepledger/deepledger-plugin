---
description: Smart transaction categorization using AI memory and historical patterns
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [vendor name | transaction description | --scan]
---

Auto-categorize transactions using learned vendor patterns. Looks up agent
memory for known vendors, checks transaction history for prior categorizations,
and proposes the correct account with confidence scores.

If $ARGUMENTS is a vendor name or transaction description, categorize that
specific transaction/vendor. If $ARGUMENTS is "--scan" or empty, scan recent
uncategorized transactions and batch-categorize them.

## Steps for Single Transaction/Vendor

Execute ALL autonomously:

1. **Look up memory**: Use agentMemory with category="categorization" and
   subject matching the vendor name to find any stored category rule
2. **Fetch master data**: Use qbMasterData to find accounts and confirm the
   vendor exists in QB
3. **Scan history**: Use qbFetchTransactions to find past transactions for
   this vendor — look at the last 10–20 transactions to identify the
   predominant account used
4. **Determine category** with confidence:
   - Memory hit (human-approved): 95% confidence
   - History consistent (same account 80%+ of the time): 90% confidence
   - History mixed (multiple accounts used): 70% confidence, show options
   - No history: 50% confidence, infer from vendor name/description using
     common sense (e.g. "AWS" → Cloud/Software, "Delta" → Travel, "Costco"
     → Office Supplies)
5. **Present result**:
   - Vendor/Description
   - Suggested Account (AcctNum + Name)
   - Confidence % with source ("from memory", "from history", "inferred")
   - If confidence < 70%, show 2–3 alternative accounts
6. **Update memory**: If confidence ≥ 90% and no existing memory, save the
   categorization pattern via agentMemory (write) so future transactions
   auto-categorize without lookup

## Steps for Bulk Scan (--scan)

1. Use qbFetchTransactions to pull the last 30 days of expenses and bills
2. Group transactions by vendor name
3. For each unique vendor, run the single-transaction lookup above (steps 1–4)
4. Present a summary table:
   | Vendor | Suggested Account | Confidence | Source |
   |--------|------------------|-----------|--------|
5. Highlight:
   - Vendors where the current account differs from the suggested one
     (possible miscategorization)
   - Vendors with low confidence that need human input
   - Any vendor using a "catch-all" account (e.g. "Miscellaneous", "Ask My
     Accountant") that could be more specifically categorized
6. For any miscategorization found (>$500 difference in impact), flag it and
   offer to create a correcting journal entry
7. Update agentMemory for all high-confidence patterns discovered

## Output Format

For single lookup:
```
Vendor: [Name]
Suggested: [AcctNum] [Account Name]
Confidence: [XX%] — [source]
[If mixed: Alternatives: AcctNum1 Name1, AcctNum2 Name2]
[Memory note: "Saved to memory" or "Updated memory"]
```

For bulk scan:
- Summary table of all vendors with categories and confidence
- Flagged items requiring attention
- Count of patterns saved to memory
- Estimate of time saved vs manual categorization
