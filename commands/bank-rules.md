---
description: Manage bank feed auto-categorization rules
allowed-tools: ["mcp__playwright__*", "mcp__plugin_deepledger_deepledger__*"]
argument-hint: <create|list|edit|delete> [details]
---

Manage QuickBooks bank rules for auto-categorizing bank feed transactions: "$ARGUMENTS"

**Important:** Bank rules are NOT available via the QuickBooks API.
This command uses browser automation to interact with the QBO UI directly.

**Prerequisites:** User must be logged into QuickBooks Online in their browser.

## Operations

### List Bank Rules
1. Navigate to Banking → Rules
2. Take screenshot
3. Present: Rule Name, Conditions, Action (category/vendor assigned)

### Create Bank Rule
1. Parse user's request for: condition (bank text, amount range) and
   action (assign to account, set vendor)
2. Verify accounts and vendors via API (qbMasterData) before setting up
3. Navigate to Banking → Rules → New Rule
4. Fill in:
   - Rule name (auto-generate: "Auto-categorize [description]")
   - Applies to: Money in / Money out
   - Conditions: Bank text contains [keyword], Amount is [range]
   - Action: Categorize to [account], Assign [vendor]
5. Screenshot for confirmation
6. On approval, save

### Example Rules

| Bank Text Contains | Category | Vendor | Rule Name |
|-------------------|----------|--------|-----------|
| "UBER" or "LYFT" | Travel | Uber/Lyft | Auto-categorize rideshare |
| "AWS" or "AMAZON WEB" | Software/SaaS | Amazon Web Services | Auto-categorize AWS |
| "GUSTO" | Payroll | Gusto | Auto-categorize payroll |
| "LANDLORD NAME" | Rent | Landlord | Auto-categorize rent |

**Tip:** After creating rules, suggest the user reviews their bank feed
(Banking tab) to see the rules applied to existing unmatched transactions.
