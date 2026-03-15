# Account Categorization — Rules and Patterns

## Categorization Consistency

The most important rule: categorize consistently. If "Staples" purchases
have been going to "Office Supplies" (AcctNum 6000), new Staples purchases
should also go to "Office Supplies" unless the user explicitly says otherwise.

## How to Maintain Consistency

### Step 1: Check Recent History

Before proposing an account for a transaction, fetch the vendor/customer's
recent transactions via qbFetchTransactions. Look at which accounts were
used in the last 3-5 transactions.

### Step 2: Follow the Pattern

If the last 3 transactions for "Staples" all used "Office Supplies":
- Default to "Office Supplies" in the proposal
- Show the user: "Based on previous transactions, categorizing to
  Office Supplies (6000). Change?"

### Step 3: Ask When Uncertain

If there's no history, or if history is inconsistent, ask the user.
Never guess. Show them the available account options from qbMasterData.

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
