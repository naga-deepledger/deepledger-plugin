---
description: Create an estimate or quote for a customer
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: <description of estimate>
---

Create a customer estimate/quote based on: "$ARGUMENTS"

Use when the user wants to send a preliminary price proposal before invoicing.
Estimates are non-binding and can be converted to invoices once accepted.

Steps:
1. Parse the description for: customer, items/services, amounts, date
2. Look up customer via qbMasterData — auto-select if single/obvious match
3. Look up items or service accounts — use history if available
4. Check for duplicate estimates via qbFetchTransactions
5. If all data is clear, execute immediately via qbEstimate with:
   - customerId, txnDate, expirationDate (default 30 days)
   - lines: [{amount, description, itemId/quantity/unitPrice}]
   - Optional: billEmail, customerMemo, salesTermId
6. Report success with estimate ID

If customer match is ambiguous or amount is missing, flag for review.

**Status tracking:** Estimates support status: Pending, Accepted, Closed,
Rejected. Update via qbEstimate with estimateId and txnStatus.

**Conversion:** When an estimate is accepted, automatically convert it to an
invoice by creating the invoice with matching line items via qbInvoice.
