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
4. Check for duplicate estimates silently
5. Propose the estimate with:
   - Customer name and ID
   - Date (default today) and expiration date (default 30 days)
   - Line items with descriptions, quantities, rates, amounts
   - Total
6. On confirmation, create via qbBill or qbInvoice tools depending on
   available MCP tools. If a dedicated estimate tool exists (qbEstimate),
   use that.

**Note:** The QBO API fully supports Estimates (full CRUD). If the MCP
server exposes an estimate tool, use it directly. If not, inform the user
that estimates can be created in the QBO UI, and offer to create a draft
invoice marked "ESTIMATE — DO NOT PAY" in the memo instead.

**Conversion:** When an estimate is accepted, offer to convert it to an
invoice: "This estimate was accepted — shall I create an invoice from it?"
