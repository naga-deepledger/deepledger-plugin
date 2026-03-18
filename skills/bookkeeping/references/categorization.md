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

### Step 2: Infer from Vendor Name and Description

For new vendors with no history, use intelligent inference:

| Vendor Name Pattern | Auto-Assign Category |
|---------------------|---------------------|
| Staples, Office Depot, Office Max | Office Supplies |
| Amazon (unless context says otherwise) | Office Supplies |
| Uber, Lyft, taxi services | Travel |
| Airlines, Hotels, Airbnb | Travel |
| Restaurants, DoorDash, Grubhub | Meals & Entertainment |
| Google Ads, Facebook Ads, ad agencies | Advertising/Marketing |
| AWS, Azure, Google Cloud, Heroku | Software/SaaS |
| Adobe, Slack, Zoom, GitHub, Notion | Software/SaaS |
| AT&T, Verizon, Comcast, internet providers | Utilities |
| Electric co, Gas co, Water utility | Utilities |
| Insurance carriers, State Farm, Geico | Insurance |
| Law firms, CPA firms, consultants | Professional Services |
| Payroll providers, ADP, Gusto | Payroll |
| Landlord, property management, "rent" | Rent/Lease |

If the vendor name or transaction description clearly matches one of these
patterns, auto-assign without asking. Note in proposal: "(inferred from
vendor name)"

### Step 3: Infer from Transaction Description

If the vendor name isn't recognizable, look at the description/memo:

- Contains "supplies", "office" → Office Supplies
- Contains "software", "subscription", "license" → Software/SaaS
- Contains "travel", "flight", "hotel" → Travel
- Contains "meal", "lunch", "dinner", "food" → Meals & Entertainment
- Contains "insurance" → Insurance
- Contains "legal", "attorney", "consulting" → Professional Services
- Contains "advertising", "marketing", "ad spend" → Advertising/Marketing
- Contains "rent", "lease" → Rent/Lease
- Contains "utility", "electric", "gas", "water", "internet" → Utilities

### Step 4: Ask Only If Genuinely Ambiguous

If NONE of the above steps yield a clear answer, then ask. Show the user
the available accounts from qbMasterData with AcctNum and let them choose.

Frame it concisely: "I can't determine the right category for this $X
payment to [vendor]. Which account should I use?" then list the top 5
most likely options.

## Categorization Consistency

The most important rule: categorize consistently. If "Staples" purchases
have been going to "Office Supplies" (AcctNum 6000), new Staples purchases
MUST go to "Office Supplies" unless the user explicitly says otherwise.

## Common Account Mappings

These are typical mappings for small businesses. Always verify against
the specific company's chart of accounts via qbMasterData.

### Expense Accounts (typical)
| Category | Common Vendors | Typical AcctNum Range |
|----------|----------------|----------------------|
| Office Supplies | Staples, Office Depot, Amazon | 6000-6099 |
| Rent/Lease | Landlord, Property Management | 6100-6199 |
| Utilities | Electric co, Gas co, Water, Internet | 6200-6299 |
| Insurance | Insurance carriers | 6300-6399 |
| Professional Services | Lawyers, CPAs, Consultants | 6400-6499 |
| Software/SaaS | Tech vendors | 6500-6599 |
| Travel | Airlines, Hotels, Uber | 6600-6699 |
| Meals & Entertainment | Restaurants | 6700-6799 |
| Advertising/Marketing | Google, Facebook, agencies | 6800-6899 |
| Payroll | Payroll providers | 6900-6999 |

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
