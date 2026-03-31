---
description: Vendor spend analysis — who you're paying, trends, and anomalies
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period or vendor name]
---

Analyze vendor spending autonomously. Pull all data upfront and present a
complete analysis with trend detection in one response — no intermediate
questions.

If $ARGUMENTS contains a vendor name, focus on that vendor. If it contains
a period, use it as the reporting period. Default to current month vs prior
month for trend analysis.

## Steps (execute ALL autonomously)

**Phase 1: Pull Data**
1. Run qbReports "VendorExpenses" for the requested period (current month)
2. Run qbReports "VendorExpenses" for prior month (comparison period)
3. Run qbReports "VendorExpenses" summarized by Month for the past 3 months
   — this is the trend data (use summarize_column_by="Month")
4. Load agentMemory (category="vendor") to check for known vendor baselines
   and previously flagged patterns

**Phase 2: Compute Metrics**

For each vendor in the current period:
- **Total spend this period** and **% of total expenses**
- **Change vs prior month** ($ amount and % change)
- **3-month trend**: Increasing / Decreasing / Stable / New (based on Phase 1
  monthly data)
- **Trend severity**: if spend increased >25% month-over-month for 2+ months,
  flag as "Accelerating"
- **Category**: pull from agentMemory or infer from name

**Phase 3: Anomaly Detection (automatic)**

Flag any of the following without asking:
- **Concentration risk**: single vendor >20% of total expenses
- **Accelerating spend**: >25% MoM increase for 2+ consecutive months
- **Spike**: >2x the vendor's typical spend this month (compare vs 3-month avg)
- **New vendor**: no history in prior periods, spend >$500
- **Possible missed payment**: vendor present last month but zero spend this
  month and is a known recurring vendor (check agentMemory)
- **Category mismatch**: same vendor switching accounts between periods

**Phase 4: Vendor Deep-Dive (if specific vendor requested)**

If a vendor name is provided:
- qbFetchTransactions for that vendor (last 90 days, all transaction types)
- Identify recurring vs one-time transactions
- Show month-over-month spend for each of the last 3 months
- Account categorization breakdown
- Flag any duplicate charges (same amount within 7 days)
- Check agentMemory for vendor-specific notes

**Phase 5: Update Memory**

For vendors with clear spending patterns (consistent category, stable or
predictable amounts), update agentMemory with:
- Category rule (vendor → account)
- Typical spend range (min/max from the 3 months of data)
- Frequency (monthly recurring vs irregular)

Only write memory if confidence ≥ 85% based on consistency of data.

## Output Format

```
VENDOR SPEND ANALYSIS — [Period]
Total Expenses: $X,XXX  |  Vendors: N  |  vs Prior: +/-$X (+/-X%)

TOP VENDORS BY SPEND
Rank | Vendor              | This Period | Prior Period | Change   | Trend
-----|---------------------|-------------|--------------|----------|--------
  1  | [Vendor Name]       | $X,XXX      | $X,XXX       | +$XXX    | Stable
  2  | [Vendor Name]       | $X,XXX      | —            | New      | New
  ...

FLAGS REQUIRING ATTENTION
⚠️  [Vendor]: Spend increased 45% for 2 consecutive months (Accelerating)
🔴  [Vendor]: 31% of total expenses (Concentration Risk)
🆕  [Vendor]: New vendor, $1,200 spend — no prior history
✅  No anomalies detected for [X] vendors

RECOMMENDATIONS
1. [Most urgent item with specific action]
2. [Second item]
...
```

For vendor deep-dive, add transaction-level table with dates, amounts, accounts.

**Phase 6: Save Vendor Spend Snapshot to Agent Memory**

After presenting results, save a vendor spend analysis snapshot to agent memory so
the portal dashboard can display vendor intelligence without re-running the command:

```
agentMemory(
  operation: "write" (or "update" if memoryId exists),
  category: "vendor_spend_analysis",
  subject: "latest_analysis",
  memory: "<JSON string: {
    \"report_date\": \"<ISO date>\",
    \"period\": \"<e.g. 2026-03>\",
    \"comparison_period\": \"<e.g. 2026-02>\",
    \"total_expenses\": <number>,
    \"total_expenses_prior\": <number>,
    \"expense_change_pct\": <number>,
    \"vendor_count\": <number>,
    \"top_vendors\": [
      {
        \"name\": \"<vendor>\",
        \"amount\": <number>,
        \"pct_of_total\": <number>,
        \"change_pct\": <number | null>,
        \"trend\": \"Increasing\" | \"Decreasing\" | \"Stable\" | \"New\"
      }
    ],
    \"flags\": [
      {
        \"type\": \"concentration_risk\" | \"accelerating\" | \"spike\" | \"new_vendor\" | \"missed_payment\" | \"category_mismatch\",
        \"vendor\": \"<name>\",
        \"detail\": \"<short description>\"
      }
    ],
    \"flag_count\": <number>,
    \"recommendations\": [\"<action item>\", ...]
  }>",
  confidence: 90
)
```

If a memory with category="vendor_spend_analysis" and subject="latest_analysis" already
exists, use operation: "update" with the memoryId to overwrite it.

Limit `top_vendors` to the top 10 by spend amount to keep the snapshot compact.
