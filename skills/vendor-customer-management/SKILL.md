---
name: vendor-customer-management
description: Manage vendors and customers in QuickBooks — create, update, lookup, analyze spending/revenue patterns, and maintain agent memory mappings. Use when the user mentions vendors, customers, payees, clients, 1099, or contact management.
---

# Vendor & Customer Management Skill

Create, update, and look up vendors and customers in QuickBooks Online. Maintain agent memory mappings for intelligent categorization. Analyze vendor spending and customer revenue patterns.

## Trigger

Activate when the user wants to:
- Create or update a vendor or customer
- Look up vendor/customer details
- Review vendor spending patterns
- Analyze customer revenue concentration
- Check 1099 vendor status
- Clean up duplicate or inactive vendors/customers
- Review agent memory mappings

## Workflow: Lookup

### Quick Lookup
```
qbMasterData(entityTypes=["vendor"]) → all vendors (name + ID)
qbMasterData(entityTypes=["customer"]) → all customers (name + ID)
qbMasterData(detailedInfo="vendor", filter="name") → full vendor details
qbMasterData(detailedInfo="customer", filter="name") → full customer details
```

### With Memory Context
```
agentMemory(operation="read", type="vendor", name="vendor name") → account mapping, upvotes, amount range
agentMemory(operation="read", type="customer", name="customer name") → income account, patterns
```

## Workflow: Create Vendor

1. **Check existing** — `qbMasterData(detailedInfo="vendor", filter="name")` to avoid duplicates
2. **Create** — `qbMasterData(operation="create", entityType="vendor", name="Vendor Name")`
3. **Optional fields**:
   - `address` — billing address
   - `vendor1099: true` — if eligible for 1099 reporting
   - `taxIdentifier` — SSN or EIN for 1099 vendors
4. **Seed memory** — `agentMemory(operation="write", type="vendor")` with suggested account mapping
5. **Confirm** — Show the created vendor with ID

## Workflow: Create Customer

1. **Check existing** — `qbMasterData(detailedInfo="customer", filter="name")` to avoid duplicates
2. **Create** — `qbMasterData(operation="create", entityType="customer", name="Customer Name")`
3. **Optional fields**:
   - `address` — billing address
4. **Seed memory** — `agentMemory(operation="write", type="customer")` with suggested income account
5. **Confirm** — Show the created customer with ID

## Workflow: Update Vendor/Customer

1. **Lookup** — `qbMasterData(detailedInfo="vendor"/"customer", filter="name")` to get current `id` and `syncToken`
2. **Update** — `qbMasterData(operation="update", entityType="vendor"/"customer", id=..., syncToken=..., field=value)`
3. **Updatable fields**: name, address, active status, 1099 eligibility, tax identifier

## Workflow: Vendor Spending Analysis

1. **Pull report** — `qbReports(reportType="VendorExpenses", startDate, endDate)` for total spend by vendor
2. **Compare periods** — Pull current vs prior period
3. **Identify**:
   - Top 10 vendors by total spend
   - Vendors with spend up > 20% vs prior period
   - New vendors (first appearance in current period)
   - Inactive vendors (no activity in 6+ months)
4. **Cross-reference memory** — Check which vendors have high vs low confidence mappings
5. **Present** — Summary with actionable insights

## Workflow: Customer Revenue Analysis

1. **Pull reports**:
   - `qbReports(reportType="SalesByCustomer", startDate, endDate)` for revenue by customer
   - `qbReports(reportType="CustomerIncome", startDate, endDate)` for customer profitability
2. **Calculate**:
   - Revenue concentration — top customer as % of total revenue
   - Customer growth — revenue change vs prior period
   - DSO by customer — who pays slowly
3. **Flag**:
   - Single customer > 30% of revenue → concentration risk
   - Customer revenue declining 3+ months → relationship issue
   - DSO increasing → collection follow-up needed

## Workflow: 1099 Review

At year-end, review vendors for 1099 eligibility:

1. **Pull vendors** — `qbMasterData(detailedInfo="vendor")` → filter where `vendor1099=true`
2. **Check spend** — `qbReports(reportType="VendorExpenses")` for each 1099 vendor
3. **Threshold** — Vendors paid $600+ need a 1099
4. **Flag missing**:
   - Vendors paid $600+ but NOT marked as 1099 eligible
   - Vendors marked 1099 but missing tax identifier
5. **Present** — Summary table for CPA review

## Workflow: Cleanup Duplicates

1. **Pull all** — `qbMasterData(entityTypes=["vendor"])` or `["customer"]`
2. **Identify** — Similar names (fuzzy match), same address, same tax ID
3. **Present** — Potential duplicates with transaction counts for each
4. **Deactivate** — `qbMasterData(operation="update", active=false)` on the duplicate (keep the one with more history)
5. **Note** — Cannot merge in QB via API; deactivating is the safe approach

## Workflow: Review Agent Memory Mappings

1. **Read all** — `agentMemory(operation="read", type="vendor")` and `type="customer"`
2. **Present** — Table with: name, mapped account, upvotes, source (bootstrap vs real-time)
3. **Flag** — Low-confidence mappings (1-2 upvotes), conflicting mappings, stale entries
4. **Allow corrections** — CPA can update any mapping via `agentMemory(operation="update")`

## Safety Checklist

- [ ] Check for existing vendor/customer before creating (avoid duplicates)
- [ ] Include `vendor1099` flag for contractors and service providers
- [ ] Seed agent memory for new vendors/customers
- [ ] User confirmation before creating or deactivating
- [ ] Never delete vendors/customers — deactivate instead

## Common Mistakes to Avoid

- Creating a duplicate vendor when one already exists (check first)
- Forgetting 1099 eligibility flag for contractors
- Missing tax identifier on 1099 vendors (needed for filing)
- Deleting instead of deactivating — loses transaction history links
- Not seeding agent memory for new vendors → next bank feed won't match
- Ignoring inactive vendors with outstanding balances
