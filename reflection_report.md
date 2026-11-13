# Munder Difflin multi-agent reflection

## System

The implementation uses five deterministic agents. The Orchestrator extracts item
quantities and dates, then calls Inventory, Quote, Fulfillment, and Reporting in
that order. Inventory maps common customer wording to the seeded catalog, checks
stock and supplier lead time, and can authorize a shortage purchase only when the
deadline and cash balance permit it. Quote uses historical quote search and applies
5% or 10% bulk discounts. Fulfillment is the only agent that writes sales or
stock-order transactions. Reporting invokes inventory, cash, and financial-report
helpers after each processed request. The system uses explicit text handoffs and
does not require a live model call, so the supplied evaluation script is
reproducible offline.

## Framework decision

I selected `smolagents` because its `@tool` decorator gives each worker a clear,
inspectable tool boundary without requiring a live model for deterministic
evaluation. The five Python agent classes perform orchestration explicitly, while
the decorated tools delegate to the supplied database helpers.

## Starter helper review

- `generate_sample_inventory`: deterministically selects stocked items and assigns
  starting quantities and reorder thresholds.
- `init_database`: creates all SQLite tables and seeds requests, quote history,
  inventory, cash, and stock transactions.
- `create_transaction`: appends validated sales or supplier stock transactions.
- `get_all_inventory`: returns positive net stock across all items at a cutoff date.
- `get_stock_level`: returns the net stock for one named item at a cutoff date.
- `get_supplier_delivery_date`: estimates lead time from quantity tiers.
- `get_cash_balance`: computes sales revenue less supplier purchase costs.
- `generate_financial_report`: combines cash, inventory valuation, and top sales.
- `search_quote_history`: finds prior quotes matching customer-language terms.

## Evaluation

Running `python project_starter.py` initializes the database, processes
`quote_requests_sample.csv` chronologically, and writes `test_results.csv`.
The evaluation processes all 20 requests. The seeded data produces at least five
fulfilled orders and multiple explicit `UNFULFILLED` responses for missing
catalog items, insufficient stock, or impossible deadlines. Successful orders
record sales revenue, so the cash balance changes for more than three requests.
Each rejection includes a reason, while successful responses include the item
list, discount-aware total, and delivery date. Supplier purchases are dated on
their estimated arrival and customer sales are dated on the resulting
fulfillment date, preventing sales before replenishment arrives. Inventory checks
also project through each promised deadline, including already committed
arrivals and sales, preventing multiple orders from reserving the same future
stock.

## Improvements

1. Replace the conservative alias table with a reviewed product taxonomy and a
   human approval queue for ambiguous item names.
2. Add transactional locking and atomic database transactions so concurrent
   orders cannot oversell the same stock.
3. Add a supplier-order table with cancellation and partial-shipment status,
   rather than recording replenishment directly as a completed stock order.
