# Sales Tax Handling Guide

## When Sales Tax Applies

Sales tax must be considered on:
- **Customer invoices** (qbInvoice) — when selling taxable goods/services
- **Sales receipts** (qbSalesReceipt) — when selling taxable goods/services
- **Vendor bills and expenses** — tax paid on purchases (input tax)

Sales tax does NOT apply to:
- Transfers between own accounts
- Journal entries (adjustments)
- Bill payments (tax was on the original bill)
- Receive payment (tax was on the original invoice)
- Most service-based businesses (varies by jurisdiction)

## Autonomous Tax Handling

### For Sales (Invoices & Sales Receipts)

1. **Check if the company collects sales tax:**
   - Look up tax codes via qbMasterData (type: "TaxCode")
   - If no tax codes exist → company likely doesn't collect sales tax → skip
   - If tax codes exist → proceed with tax application

2. **Determine if the transaction is taxable:**
   - Physical goods → usually taxable
   - Digital goods → varies by state
   - Services → usually NOT taxable (but varies)
   - If user says "plus tax" or mentions tax → apply tax
   - If unclear and company has tax codes → ask: "Should this include sales tax?"

3. **Apply the correct tax code:**
   - Use the company's default tax code if only one exists
   - If multiple tax codes → match by customer's shipping state if possible
   - Include taxCodeId on each taxable line item

4. **Show tax in proposal:**
   ```
   Subtotal:    $1,000.00
   Sales Tax:   $   82.50  (8.25% — TX Sales Tax)
   Total:       $1,082.50
   ```

### For Purchases (Expenses & Bills)

1. Tax paid on purchases is typically included in the total amount
2. Record the full amount (including tax) to the expense account
3. Do NOT separate purchase tax unless the user specifically requests it
   (some businesses track input tax for tax credit purposes)
4. If user says "before tax" or gives a pre-tax amount:
   - Ask for tax rate or look up applicable rate
   - Calculate total and show in proposal

### Tax Rate Lookup

Use qbMasterData to retrieve tax rates:
- type: "TaxCode" — returns available tax codes
- Each tax code has a rate percentage and applicable jurisdiction

If no tax rates are configured in QuickBooks:
- Do NOT guess tax rates
- Inform the user: "No tax rates are configured in QuickBooks. Would you
  like me to help set one up, or should I record this without tax?"

## Common Scenarios

| Scenario | Tax Handling |
|----------|-------------|
| "Invoice client $5,000 for consulting" | No tax (service) unless user says otherwise |
| "Invoice for 100 widgets at $10 each" | Apply tax (physical goods) — use default rate |
| "Sold merchandise for $500 plus tax" | Apply tax — user explicitly requested it |
| "Paid $1,082.50 to supplier including tax" | Record full amount as expense |
| "Bought supplies for $200 before tax" | Ask for or look up tax rate, calculate total |

## Tax Remittance

During month-end close (/close-books), check:
1. Total sales tax collected this period
2. Whether a tax remittance entry exists
3. If tax is due, suggest: "You collected $X in sales tax this period.
   Has this been remitted to the tax authority? If not, the liability
   is sitting in your Sales Tax Payable account."

## Tax-Exempt Transactions

Some customers or transactions are tax-exempt:
- Non-profit organizations (with exemption certificate)
- Resale purchases (with resale certificate)
- Interstate sales (varies by nexus)
- Government agencies

If a customer has been historically invoiced without tax, continue that
pattern. If the user mentions "tax exempt," note it in the invoice memo
and do not apply tax.
