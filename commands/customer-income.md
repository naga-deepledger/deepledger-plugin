---
description: Customer revenue intelligence — top customers, late payers, churn risk, and payment velocity
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: [period or customer name]
---

Analyze customer revenue autonomously. Pull all data and present complete
intelligence with payment behavior analysis in one response — no intermediate
questions.

If $ARGUMENTS contains a customer name, focus on that customer. If it
contains a period, use it as the reporting period. Default to current month.

## Steps (execute ALL autonomously)

**Phase 1: Pull Data (in parallel where possible)**
1. qbReports "CustomerIncome" for the requested period (current month)
2. qbReports "CustomerIncome" for the comparison period (prior month)
3. qbReports "CustomerIncome" summarized by Month for the past 3 months
   (use summarize_column_by="Month") — trend data
4. qbReports "AgedReceivablesDetail" as of today — payment behavior data
5. qbReports "AgedReceivables" as of today — summary AR aging buckets
6. Load agentMemory (category="customer") for known customer patterns
7. qbFetchTransactions(transactionType: "Payment", startDate: <90-days-ago>,
   endDate: <today>, maxResults: 500) — recent payment history

**Phase 2: Revenue Analysis**

For each customer in the current period, compute:
- **Total revenue** this period and % of total company revenue
- **Change vs prior period** ($ amount and % change)
- **3-month trend**: Increasing / Decreasing / Stable / New / Lost
- **Trend velocity**: average MoM growth rate over 3 months
- **Customer lifetime**: first seen date (from agentMemory or oldest transaction)

Rank customers by total revenue, highest first.

**Phase 3: Payment Behavior Intelligence**

For each significant customer (top 10 by revenue OR any with AR > $500):

A. **Payment Velocity Score** (1-10 scale):
   Analyze AgedReceivablesDetail to determine how quickly each customer pays.

   Scoring:
   - All invoices paid within terms (Current bucket only): **10** (Excellent)
   - Mostly current, <10% in 1-30 bucket: **8** (Good)
   - 10-25% in 1-30 bucket: **6** (Fair)
   - >25% in 1-30 or any in 31-60: **4** (Slow)
   - Any amount in 61-90 bucket: **2** (Very Slow)
   - Any amount in 91+ bucket: **1** (Critical — collections needed)

B. **Late Payer Detection**:
   From AgedReceivablesDetail, identify customers with:
   - Any invoice >30 days overdue → flag as "Late Payer"
   - Any invoice >60 days overdue → flag as "Chronic Late Payer"
   - Any invoice >90 days overdue → flag as "Collections Risk"
   - Pattern of getting slower (compare current AR aging to agentMemory) → "Deteriorating"

C. **Days Sales Outstanding (DSO) Estimate** per customer:
   If payment transactions available: average days between invoice date and
   payment date from recent Payment transactions.
   Otherwise estimate from AR aging buckets.

D. **Outstanding Balance**: Total AR owed by this customer right now.

**Phase 4: Risk Alerts (always include)**

Automatically flag — do not ask:

🔴 **CRITICAL Alerts**:
- Customer with >30% of total revenue (extreme concentration risk)
- Customer with AR >90 days overdue AND >$5,000 (write-off risk)
- Top 3 customer declining >40% MoM for 2+ months (major churn risk)

🟡 **WARNING Alerts**:
- Customer with >20% of total revenue (concentration risk)
- Customer with payment velocity score ≤ 4 AND outstanding AR >$1,000
- Revenue from top 3 customers >70% of total (portfolio concentration)
- Customer declining >25% MoM (churn risk)
- New customer with single large invoice >$5,000 (verify legitimacy)

🟢 **POSITIVE Signals**:
- Customer with >25% MoM growth for 2+ months (expanding relationship)
- New customer added this period (growth)
- Customer improved payment velocity vs prior quarter

**Phase 5: Customer Deep-Dive (if specific customer requested)**

If a customer name is provided in $ARGUMENTS:
1. qbFetchTransactions for that customer (Invoice, Payment — last 180 days)
2. Build invoice-by-invoice timeline:
   - Invoice date → amount → payment date → days to pay
3. Calculate: average invoice size, median days to pay, payment consistency
4. Show outstanding invoices with aging
5. Check agentMemory for customer-specific notes and prior flags
6. Revenue trend chart data (last 6 months if available)

**Phase 6: Save Customer Intelligence to Agent Memory**

After analysis, persist a customer intelligence snapshot to agentMemory so
the portal dashboard and other commands can access it:

```
agentMemory(
  operation: "write",
  category: "customer_intelligence",
  subject: "latest_snapshot",
  memory: "<JSON string: {
    \"snapshot_date\": \"<ISO date>\",
    \"period\": \"<e.g. 2026-03>\",
    \"total_revenue\": <number>,
    \"customer_count\": <N>,
    \"top_customers\": [
      {\"name\": \"<name>\", \"revenue\": <N>, \"pct_of_total\": <N>, \"trend\": \"<Increasing|Decreasing|Stable>\", \"payment_velocity\": <1-10>, \"outstanding_ar\": <N>}
    ],
    \"concentration_risk\": {\"top3_pct\": <N>, \"max_single_pct\": <N>, \"status\": \"<safe|warning|critical>\"},
    \"late_payers\": [{\"name\": \"<name>\", \"outstanding\": <N>, \"oldest_invoice_days\": <N>, \"velocity_score\": <1-10>}],
    \"churn_risks\": [{\"name\": \"<name>\", \"decline_pct\": <N>, \"months_declining\": <N>}],
    \"growth_signals\": [{\"name\": \"<name>\", \"growth_pct\": <N>, \"months_growing\": <N>}],
    \"total_ar_outstanding\": <N>,
    \"avg_payment_velocity\": <1-10>
  }>",
  confidence: 90
)
```

If an entry for category="customer_intelligence", subject="latest_snapshot"
already exists, use operation: "update" with the memoryId.

Also update per-customer memory for significant customers:
```
agentMemory(
  operation: "write",
  category: "customer",
  subject: "<customer name>",
  memory: "<JSON string: {
    \"typical_monthly_revenue\": <N>,
    \"payment_velocity\": <1-10>,
    \"avg_days_to_pay\": <N>,
    \"trend\": \"<Increasing|Stable|Decreasing>\",
    \"last_invoice_date\": \"<date>\",
    \"total_ar\": <N>,
    \"notes\": \"<any flags or observations>\"
  }>",
  confidence: 85
)
```

**Phase 7: Present Results**

Output format:

```
CUSTOMER REVENUE INTELLIGENCE — [Period]
Total Revenue: $X,XXX  |  Customers: N  |  vs Prior: +/-$X (+/-X%)

TOP CUSTOMERS
Rank | Customer            | Revenue    | vs Prior   | Trend       | Payment | AR Owed
-----|---------------------|------------|------------|-------------|---------|--------
  1  | [Name]              | $X,XXX     | +$X (+X%)  | Increasing  | 8/10    | $X,XXX
  2  | [Name]              | $X,XXX     | -$X (-X%)  | Decreasing  | 3/10    | $X,XXX
  ...

PAYMENT BEHAVIOR
⚡ Fast Payers (8-10): [Customer1], [Customer2]
🐢 Slow Payers (4-6): [Customer3] ($X,XXX overdue)
🚨 Collections Risk (1-3): [Customer4] ($X,XXX, 95 days overdue)

CONCENTRATION RISK
Top 3 customers = X% of revenue [SAFE / WARNING / CRITICAL]
Largest single customer: [Name] at X%

ALERTS
🔴 [Customer]: $X,XXX AR >90 days — recommend collections action
🟡 [Customer]: Revenue declining 3 consecutive months (-35% total)
🟢 [Customer]: Revenue growing 40% MoM — expanding relationship

RECOMMENDATIONS
1. [Most urgent: specific action with customer name and dollar amount]
2. [Second priority action]
3. [Collection or diversification recommendation]
```

For customer deep-dive, add:
- Invoice-level table with dates, amounts, payment dates, days-to-pay
- Payment consistency analysis
- Recommended actions specific to this customer

**Tone:** Data-driven, specific dollar amounts, actionable. Frame findings
as business intelligence, not just numbers. Answer: "Who should I follow up
with?" and "Where is revenue at risk?"
