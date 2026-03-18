# Account Categorization — Autonomous Rules and Patterns

## Core Principle: Categorize Autonomously

The goal is zero-question categorization for the vast majority of
transactions. Only ask the user when the correct account is genuinely
ambiguous AND getting it wrong would misstate financials.

## Autonomous Categorization Decision Tree

### Step 1: Check Vendor/Customer History (ALWAYS do this first)

Fetch the entity's recent transactions via qbFetchTransactions. Look at
which accounts were used in the last 3-5 transactions.

- **Consistent history (3+ transactions, same account):**
  Use that account. Do NOT ask. Do NOT even mention it — just include it
  in the proposal as the selected account.

- **Mixed history (different accounts used):**
  Use the most frequently used account. Note in proposal:
  "(based on most common categorization for [vendor])"

- **No history (new vendor/customer):**
  Proceed to Step 2.

### Step 2: Infer from Description + Vendor Name Together

For new vendors with no history, use BOTH the transaction description
AND the vendor name to determine the account. The description the user
provides is the strongest signal — it tells you what was actually purchased.

**Description keywords take priority over vendor name:**

| Description Contains | Category |
|---------------------|----------|
| "supplies", "office supplies", "toner", "paper" | Office Supplies |
| "software", "subscription", "license", "SaaS" | Software/SaaS |
| "hosting", "server", "cloud", "infrastructure" | Software/SaaS |
| "travel", "flight", "hotel", "airfare", "lodging" | Travel |
| "ride", "taxi", "transportation" | Travel |
| "meal", "lunch", "dinner", "food", "catering" | Meals & Entertainment |
| "insurance", "premium", "coverage", "policy" | Insurance |
| "legal", "attorney", "consulting", "advisory" | Professional Services |
| "accounting", "audit", "tax preparation" | Professional Services |
| "advertising", "marketing", "ad spend", "campaign" | Advertising/Marketing |
| "rent", "lease", "office space" | Rent/Lease |
| "electric", "gas", "water", "internet", "phone" | Utilities |
| "payroll", "wages", "salary" | Payroll |
| "repair", "maintenance", "fix" | Repairs & Maintenance |
| "training", "course", "conference", "seminar" | Training & Education |

**Then use vendor name — but only for single-purpose vendors:**

Vendors fall into two categories:

**Single-purpose vendors (safe to infer from name alone):**
These vendors sell ONE thing. The name IS the category.

| Vendor | Category | Why Safe |
|--------|----------|----------|
| Uber, Lyft | Travel | Only sell rides |
| United Airlines, Delta, Southwest | Travel | Only sell flights |
| Marriott, Hilton, Airbnb | Travel | Only sell lodging |
| DoorDash, Grubhub, UberEats | Meals & Entertainment | Only deliver food |
| Google Ads, Facebook Ads | Advertising/Marketing | Ad-specific products |
| Slack, Zoom, GitHub, Notion, Figma | Software/SaaS | Single software product |
| Adobe, Salesforce, HubSpot | Software/SaaS | Software companies |
| ADP, Gusto, Paychex | Payroll | Payroll processors |
| State Farm, Geico, Allstate | Insurance | Insurance carriers |
| AT&T, Verizon, Comcast, T-Mobile | Utilities | Telecom/internet only |
| Office Depot, Office Max, Staples | Office Supplies | Office supply stores |

**Multi-purpose vendors (NEVER infer from name alone — need description):**
These vendors sell many categories of things. The vendor name tells you
nothing about what was purchased.

| Vendor | Could Be | What To Do |
|--------|----------|------------|
| Amazon | Office Supplies, Software, Inventory, Equipment, Books | MUST use description or ask |
| Costco | Office Supplies, Meals, Inventory, Equipment | MUST use description or ask |
| Walmart | Office Supplies, Meals, Inventory, Equipment | MUST use description or ask |
| Target | Office Supplies, Meals, Equipment | MUST use description or ask |
| Best Buy | Software, Equipment, Office Supplies | MUST use description or ask |
| Google | Advertising, Software/SaaS, Cloud hosting | MUST use description or ask |
| Apple | Software, Equipment, Subscription | MUST use description or ask |
| Microsoft | Software, Cloud hosting, Equipment | MUST use description or ask |

For multi-purpose vendors: if the user's description makes the category
clear (e.g., "Amazon for office supplies"), use that. If the description
is vague (e.g., just "Amazon $150"), ask.

### Step 3: Ask Only If Genuinely Ambiguous

If NONE of the above yields a clear answer, ask. Show the user the
available accounts from qbMasterData with AcctNum and let them choose.

Frame it concisely: "I can't determine the right category for this $X
payment to [vendor]. Which account should I use?" then list the top 5
most likely options based on what you do know.

## Categorization Consistency

The most important rule: categorize consistently. If "Staples" purchases
have been going to "Office Supplies" (AcctNum 6000), new Staples purchases
MUST go to "Office Supplies" unless the user explicitly says otherwise.

**History always wins.** Even if "Amazon" is a multi-purpose vendor, if
the company's last 5 Amazon transactions all went to "Office Supplies",
the next one should default there too (with a note in the proposal).

## Common Account Mappings

These are typical mappings for small businesses. Always verify against
the specific company's chart of accounts via qbMasterData.

### Expense Accounts (typical)
| Category | Typical AcctNum Range |
|----------|----------------------|
| Office Supplies | 6000-6099 |
| Rent/Lease | 6100-6199 |
| Utilities | 6200-6299 |
| Insurance | 6300-6399 |
| Professional Services | 6400-6499 |
| Software/SaaS | 6500-6599 |
| Travel | 6600-6699 |
| Meals & Entertainment | 6700-6799 |
| Advertising/Marketing | 6800-6899 |
| Payroll | 6900-6999 |

### Income Accounts (typical)
| Category | Typical AcctNum Range |
|----------|----------------------|
| Service Revenue | 4000-4099 |
| Product Sales | 4100-4199 |
| Consulting Income | 4200-4299 |
| Other Income | 4900-4999 |

### Balance Sheet Accounts (typical)
| Category | Typical AcctNum Range |
|----------|----------------------|
| Checking/Savings | 1000-1099 |
| Accounts Receivable | 1100-1199 |
| Inventory | 1200-1299 |
| Fixed Assets | 1500-1599 |
| Accounts Payable | 2000-2099 |
| Credit Cards | 2100-2199 |
| Loans | 2500-2599 |
| Equity | 3000-3099 |

## When Account Doesn't Exist

If the appropriate account doesn't exist in the chart of accounts:
1. Suggest the closest existing account
2. Ask if user wants to create a new account via qbMasterData (operation: 'create')
3. If creating, suggest an appropriate AcctNum based on the category ranges above

## Account Types

QuickBooks account types determine where they appear on financial statements:
- **Income** → P&L (revenue section)
- **Expense** → P&L (expense section)
- **Cost of Goods Sold** → P&L (COGS section)
- **Bank** → Balance Sheet (assets)
- **Accounts Receivable** → Balance Sheet (assets)
- **Other Current Asset** → Balance Sheet (assets)
- **Fixed Asset** → Balance Sheet (assets)
- **Accounts Payable** → Balance Sheet (liabilities)
- **Credit Card** → Balance Sheet (liabilities)
- **Other Current Liability** → Balance Sheet (liabilities)
- **Long Term Liability** → Balance Sheet (liabilities)
- **Equity** → Balance Sheet (equity)
