# Vendor Baseline Model

## Overview

A vendor baseline is a structured profile stored in `agentMemory` that captures
a vendor's typical transaction behavior. Baselines are the foundation of anomaly
detection — without them, the system can only flag obvious issues like duplicates
and round numbers. WITH baselines, it can flag amount spikes, frequency changes,
category drift, and seasonal anomalies.

## Data Schema

Each baseline is stored as one `agentMemory` entry:

- **category:** `"vendor_baseline"`
- **subject:** `"baseline:<vendor_display_name>"` (lowercase, trimmed)
- **confidence:** 60 (first scan) → 75 (2+ scans) → 90 (10+ scans with human feedback)
- **memory:** JSON string with this structure:

```json
{
  "vendor_name": "Acme Corp",
  "vendor_id": "QB-vendor-id-if-known",
  "first_seen": "2025-06-15",
  "last_seen": "2026-03-01",
  "scan_count": 3,
  "transaction_count": 24,
  "amount": {
    "min": 245.00,
    "max": 1250.00,
    "mean": 687.50,
    "median": 650.00,
    "stddev": 285.30,
    "p95": 1100.00
  },
  "frequency": {
    "pattern": "monthly",
    "avg_days_between": 30,
    "min_days_between": 27,
    "max_days_between": 35,
    "transactions_per_month": 1.0
  },
  "categorization": {
    "primary_account": "Software Subscriptions",
    "primary_account_id": "123",
    "primary_pct": 100,
    "secondary_account": null,
    "secondary_pct": 0
  },
  "flags": {
    "is_recurring": true,
    "is_single_purpose": true,
    "has_round_amounts": false,
    "weekend_transactions_pct": 0,
    "seasonal_pattern": null
  },
  "anomaly_history": {
    "times_flagged": 1,
    "times_cleared": 1,
    "false_positive_rate": 1.0
  },
  "updated_at": "2026-03-01T12:00:00Z"
}
```

## Amount Thresholds (derived from baseline)

Once a baseline exists, anomaly detection uses these rules:

| Condition | Risk Level | Action |
|-----------|-----------|--------|
| amount > mean + 3×stddev | HIGH (60+) | Flag for review |
| amount > mean + 2×stddev | MEDIUM (40-59) | Flag for review |
| amount > p95 × 1.5 | MEDIUM (40-59) | Flag for review |
| amount > max × 2 | HIGH (60+) | Flag for review |
| amount < min × 0.3 | LOW (20-39) | Log only (unusually small) |

When stddev is 0 (all identical amounts): flag anything >10% different as MEDIUM.

## Frequency Thresholds

| Condition | Risk Level |
|-----------|-----------|
| Monthly vendor appears 3+ times in a month | MEDIUM |
| Weekly vendor missing >2 expected occurrences | LOW (note) |
| Vendor reappears after >3× avg_days_between | LOW (note) |
| Vendor frequency doubles suddenly | MEDIUM |

## Building Baselines

### First Scan (no existing baselines)

1. Fetch ALL transactions for the last 90 days (3 months gives good seasonality signal):
   - `qbFetchTransactions` for Purchase, Bill, Expense types
2. Group by vendor (use `VendorRef.name` or entity name)
3. For each vendor with ≥2 transactions:
   - Compute amount stats (min, max, mean, median, stddev, p95)
   - Compute frequency (avg days between, pattern detection)
   - Identify primary account categorization
   - Set flags (recurring, single-purpose, round amounts, etc.)
4. Save each baseline to agentMemory:
   ```
   agentMemory(
     operation: "write",
     category: "vendor_baseline",
     subject: "baseline:<vendor_name_lowercase>",
     memory: "<JSON string>",
     confidence: 60
   )
   ```
5. For vendors with only 1 transaction: store a minimal baseline with
   `scan_count: 1`, `transaction_count: 1`, wide amount range
   (min=amount×0.5, max=amount×2), frequency="unknown"

### Subsequent Scans (baselines exist)

1. Read existing baselines: `agentMemory(action: "read", category: "vendor_baseline")`
2. Fetch transactions since the baseline's `last_seen` date
3. For each vendor:
   - Merge new transaction data into existing stats
   - Recompute amount stats with combined dataset
   - Update frequency metrics
   - Increment scan_count and transaction_count
   - Update last_seen and updated_at
4. Update via: `agentMemory(operation: "update", memoryId: <id>, ...)`
5. New vendors get fresh baselines as in first scan

### Pattern Detection Heuristics

**Frequency classification:**
- `monthly`: avg_days_between 25-35, consistent for 3+ occurrences
- `weekly`: avg_days_between 5-9
- `biweekly`: avg_days_between 12-16
- `quarterly`: avg_days_between 80-100
- `annual`: avg_days_between 340-400
- `irregular`: anything else

**Single-purpose detection:**
- primary_pct ≥ 95% → is_single_purpose = true
- Vendor name contains category keywords (e.g., "Insurance", "Software") → true

**Recurring detection:**
- frequency != "irregular" AND transaction_count ≥ 3 → is_recurring = true

**Seasonal pattern detection:**
- Group amounts by month-of-year across multiple years
- If certain months consistently show 2x+ other months → seasonal_pattern = "Q4-spike" etc.
- Only meaningful with 12+ months of data

## Integration Points

- **Anomaly Detection Skill:** Reads baselines before scanning, uses thresholds above
- **Auto-Categorization:** Can reference baselines for vendor→account confidence
- **Self-Audit:** Baselines provide ground truth for accuracy measurement
- **/build-baselines command:** Dedicated command to create/refresh baselines
- **/anomaly-scan command:** Reads baselines as part of standard scan flow
- **Autonomous Loop:** Rebuilds baselines weekly after anomaly scan completes

## Confidence Evolution

Baseline confidence grows with validation:

| Condition | Confidence |
|-----------|-----------|
| First scan, ≥2 transactions | 60 |
| Second scan, consistent with first | 75 |
| 5+ scans, no false positives | 85 |
| 10+ scans, human-cleared anomalies | 90 |
| Human explicitly corrected a flag (false positive) | +5 to that vendor |
| Human confirmed an anomaly was real | no change (system was right) |
