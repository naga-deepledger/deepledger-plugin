---
description: Create a purchase order for a vendor
allowed-tools: ["mcp__plugin_deepledger_deepledger__*"]
argument-hint: <description of purchase order>
---

Create a vendor purchase order based on: "$ARGUMENTS"

Use when the user wants to commit to purchasing from a vendor before
receiving goods or a bill.

Steps:
1. Parse the description for: vendor, items, quantities, amounts, date
2. Look up vendor via qbMasterData — auto-select if single/obvious match
3. Look up items via qbMasterData if applicable
4. Check for duplicate POs via qbFetchTransactions
5. If all data is clear, execute immediately via qbPurchaseOrder:
   - vendorId, txnDate, lines, dueDate (if mentioned)
6. Report success

**Note:** The QBO API fully supports Purchase Orders (full CRUD). If the
MCP server exposes a qbPurchaseOrder tool, use it directly. If not, inform
the user that the PO can be created in the QBO UI, and offer to record the
bill (qbBill) when goods arrive instead.

**Receiving:** When goods arrive, offer: "This PO has been fulfilled —
shall I convert it to a bill?"
