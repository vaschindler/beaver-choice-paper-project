# Munder Difflin agent flow

```mermaid
flowchart TD
    C[Customer text inquiry] --> O[Orchestrator agent]
    O --> P[Parse items and dates]
    P --> I[Inventory agent]
    I -->|get_all_inventory / get_stock_level| DB[(SQLite)]
    I -->|shortage check| S[get_supplier_delivery_date]
    I -->|large-order health check| R[Reporting agent]
    R -->|get_cash_balance / generate_financial_report| DB
    I --> Q[Quote agent]
    Q -->|search_quote_history + prices + bulk discount| DB
    Q --> D{Stock and deadline feasible?}
    D -->|No| X[Text rejection with reason]
    D -->|Yes| F[Fulfillment agent]
    F -->|create_transaction stock_orders and sales| DB
    F --> R
    R --> A[Confirmation with total, delivery, and updated status]
```

There are five agents: Orchestrator, Inventory, Quote, Fulfillment, and Reporting.
Agents communicate through typed Python records and customer-facing text; database
access is restricted to the supplied helper functions.
