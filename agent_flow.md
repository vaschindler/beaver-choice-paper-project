# Munder Difflin agent flow

```mermaid
flowchart TD
    C[Customer text inquiry] --> O[Orchestrator agent]
    O --> P[Parse items and dates]
    P --> I[Inventory agent]
    I --> IT1[inventory_snapshot_tool<br/>get_all_inventory: all positive stock]
    I --> IT2[item_stock_tool<br/>get_stock_level: one item's stock]
    I --> IT3[supplier_delivery_tool<br/>get_supplier_delivery_date: lead time]
    I --> IT4[cash_balance_tool<br/>get_cash_balance: reorder affordability]
    IT1 --> DB[(SQLite)]
    IT2 --> DB
    IT3 --> S[delivery date]
    IT4 --> DB
    I --> Q[Quote agent]
    Q --> QT[quote_history_tool<br/>search_quote_history: comparable quotes]
    QT --> DB
    Q -->|catalog prices + bulk discount| D{Stock and deadline feasible?}
    D -->|No| X[Text rejection with reason]
    D -->|Yes| F[Fulfillment agent]
    F --> FT[transaction_tool<br/>create_transaction: supplier purchases and sales]
    FT --> DB
    F --> R
    R[Reporting agent] --> RT[financial_report_tool<br/>generate_financial_report: final health report]
    RT --> DB
    R --> A[Confirmation with total, delivery, and updated status]
```

There are five agents: Orchestrator, Inventory, Quote, Fulfillment, and Reporting.
The implementation uses the `smolagents` tool decorator. Agents communicate
through typed Python records and customer-facing text; database access is
restricted to the decorated tools, each of which delegates to a supplied starter
helper. The orchestrator delegates in this order: parse -> inventory -> quote ->
decision -> fulfillment -> reporting.
