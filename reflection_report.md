# Munder Difflin multi-agent reflection

## System

The implementation uses five deterministic agents. The Orchestrator extracts item
quantities and dates, then calls Inventory, Quote, Fulfillment, and Reporting in
that order. Inventory maps common customer wording to the seeded catalog, checks
stock and supplier lead time, and can authorize a shortage purchase only when the
deadline and cash balance permit it. Quote uses historical quote search and applies
5% or 10% bulk discounts. Fulfillment is the only agent that writes sales or
stock-order transactions. Reporting invokes inventory, cash, and financial-report
helpers after each processed request.

The system uses explicit text handoffs instead of requiring an API key or an
external model, so the supplied evaluation script is reproducible offline.

## Framework decision

I selected a lightweight Python orchestration layer for this offline evaluation
instead of `smolagents`, `pydantic-ai`, or `npcsh`. The agent boundaries and text
handoffs are explicit Python classes, which avoids making the required test run
dependent on an API key or network service while preserving the same delegation
model those frameworks would provide.

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
The seeded data produces multiple fulfilled small orders and multiple explicit
`UNFULFILLED` responses for missing catalog items, insufficient stock, or
impossible deadlines. Each rejection includes a reason, while successful
responses include the item list, discount-aware total, and delivery date.

## Improvements

1. Replace the conservative alias table with a reviewed product taxonomy and a
   human approval queue for ambiguous item names.
2. Add transactional locking and atomic database transactions so concurrent
   orders cannot oversell the same stock.
3. Add a supplier-order table with cancellation and partial-shipment status,
   rather than recording replenishment directly as a completed stock order.
