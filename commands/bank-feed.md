# /bank-feed

Process unrecorded bank and credit card transactions from the bank feed.

## Usage
```
/bank-feed                    # Process all unrecorded transactions
/bank-feed review             # Show transactions without recording
/bank-feed flag <reason>      # Flag a specific transaction for CPA review
```

## Behavior

Use the **Accountant** agent with the **Bookkeeping** skill.

1. Fetch unprocessed transactions: `bankFeed(action="fetch")`
2. For each transaction, evaluate confidence using enrichment data:
   - **High confidence** (vendor in memory, 3+ upvotes) → show and record with confirmation
   - **Medium confidence** (vendor in memory, 1-2 upvotes) → show with suggested category, ask user
   - **Low confidence** (no memory match) → flag for CPA review with reasoning
3. Before recording any transaction:
   - Check if an outstanding Bill exists → use `qbBillPayment` instead of `qbExpense`
   - Check if an outstanding Invoice exists → use `qbReceivePayment` instead of `qbDeposit`
   - Run duplicate check via `qbFetchTransactions`
4. After recording, upvote the vendor memory and mark as recorded via `fetchWorkQueue(source="markRecorded")`
5. Report summary: X recorded, Y flagged for review, Z skipped

## Review Mode
When called with `review`, show all transactions in a table format without recording any. Let the user pick which ones to process.

## Flag Mode
When called with `flag`, use `bankFeed(action="flag")` to send the specified transaction to the review queue with the provided reason as `aiReasoning`.
