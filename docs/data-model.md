# Data Model

This solution is the "buy side" counterpart to sales: how much is being received from
suppliers, of which brands/products, how that's trending against last year, and how
many receiving transactions it took to get there. It's a small, focused star schema —
one fact table hung off the same product/brand/calendar dimensions used in the
companion Power BI projects.

> **Scope note.** This repository covers **Purchase & Supply** only — incoming
> goods-receipt value, quantity, and transaction volume by brand, product, and time.
> Sales monitoring, inventory, pricing & margin, trade marketing, sales planning, and
> portfolio analytics are covered in separate, standalone Power BI projects and are
> out of scope here.

## Entity-relationship diagram

```mermaid
erDiagram
    BRANDS ||--o{ PRODUCTS : "groups"
    PRODUCTS ||--o{ GOODS_RECEIPT_LINES : "received"
    CALENDAR ||--o{ GOODS_RECEIPT_LINES : "dated"
```

## Table catalog

### Dimensions

| Table | Purpose |
|---|---|
| **Calendar** | Date dimension backing every current-year / prior-year / YTD comparison in this solution. |
| **Products** | Product master — the grain purchase activity is tracked and ranked at. |
| **Brands** | Brand master (10 brands across 4 divisions) — the grouping level for the purchase-share and brand-comparison visuals. |

### Facts

| Table | Purpose |
|---|---|
| **Goods Receipt Lines** | One row per supplier invoice line received: quantity, supply base price, standard discount %, and extra discount %. This is purchase & supply data — it does not touch customer sales, pricing to customers, or stock coverage, which are covered in separate, standalone projects. |

## Relationships

| From | To |
|---|---|
| Products[BrandId] | Brands[Id] |
| Goods Receipt Lines[ProductId] | Products[Id] |
| Goods Receipt Lines[Date] | Calendar[Date] |

## Design note: transactions, not just totals

Alongside value and quantity, this solution tracks **purchase invoice count** and
**distinct products purchased** as first-class KPIs, not just a value/quantity pair.
That's deliberate — a buyer needs to know not only *how much* was bought, but whether
it arrived as a handful of large consolidated shipments or many small, fragmented
receiving transactions, since that shapes receiving-team workload independently of
the value on the page.
