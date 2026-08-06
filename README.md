# Purchase Analytics

A **Power BI** purchase & supply monitoring solution built for a multi-brand FMCG/
consumer-goods distribution business — incoming goods-receipt value, quantity, and
transaction volume from suppliers, by brand, product, and time.

This repository documents the architecture and DAX behind this **Purchase
Analytics** solution. It's a portfolio write-up, not the live report: all entity
names in the underlying model are generic sample labels (`Product 001`, `Brand 1`…),
and no real business figures, screenshots, or connection details are included here.

## Business context

The business distributes ~10 brands across roughly 4 divisions, replenished through
regular supplier goods receipts. This solution is the buy-side complement to sales
monitoring: how much is coming in, from which brands and products, how that compares
to last year, and how many receiving transactions it took — visibility purchasing
and category teams need without cross-referencing supplier paperwork by hand.

## What this covers

- **Purchase value & quantity trend** — monthly and weekly, benchmarked against
  prior year.
- **Transaction volume, tracked as its own KPI** — invoice count and distinct
  products purchased, not just a value/quantity total, since a handful of large
  shipments and many small fragmented receipts carry very different operational
  cost even at the same total value.
- **Brand & product share** — purchase share of total by brand, and product-level
  value/quantity with a YoY growth index.

## Architecture

```mermaid
flowchart LR
    ERP[("On-prem ERP\n(SQL Server)")] -->|Import, Power Query M| SM[Semantic Model\nTMDL / PBIP]
    SM --> FN["Shared DAX function library\nYearToDate · PriorYear · GrowthIndex"]
    FN --> DAX["Purchase KPIs\nvalue · quantity · invoice count\nproducts purchased"]
    SM --> RLS["Row-Level Security\n(role + culture per user)"]
    DAX --> RPT[Report — 2 pages]
    RLS --> RPT
    RPT --> EN[English UI]
    RPT --> Alt[Second language UI\nswitched live]
```

## Tech stack

- **Power BI** — authored in [PBIP](https://learn.microsoft.com/power-bi/developer/projects/projects-overview) (TMDL) format for source control
- **DAX user-defined functions** — reuses 3 functions from the solution's shared time-intelligence library (see [DAX Functions Used](docs/dax-functions.md))
- **DAX measures** — purchase value, quantity, transaction, and share KPIs built on that library (see [DAX Measure Catalog](docs/dax-measures.md))
- **Power Query (M)** — SQL Server import and shaping
- **Row-Level Security** — role- and culture-based, resolved from a single access-control table
- **Dynamic localization** — a live culture toggle driving every page and visual title in the report from one measure

## Documentation

| Doc | Contents |
|---|---|
| [Data Model](docs/data-model.md) | Star schema, ER diagram, table catalog, relationships, RLS, and the transaction-volume design rationale |
| [DAX Functions Used](docs/dax-functions.md) | The 3 shared time-intelligence functions this solution calls |
| [DAX Measure Catalog](docs/dax-measures.md) | Purchase value, quantity, transaction-volume, and share measures |
| [Dynamic Titles & Localization](docs/dynamic-titles.md) | The `Security` → `Labels` → `Dynamic Labels` chain in full: the `PageKey` band scheme, every label family worked through, the exact report-visual binding, and what changes to add a language |
| [Report Tour](docs/report-pages.md) | Walkthrough of both pages |

## Highlights worth a closer look

- **Transactions as a first-class KPI.** Purchase invoice count and distinct
  products-purchased sit alongside value and quantity, not buried in a detail table —
  because receiving-team workload scales with transaction count, independent of
  total purchase value.
- **Consistent time-intelligence, reused, not duplicated.** Every current/prior/YTD/
  growth-index pair composes the same 3 shared functions used across the rest of the
  solution — no separate purchasing-specific date logic to maintain.
- **Titles bind to measures, never hardcoded text.** Every page title is a hidden
  action-button visual with its title/subtitle bound via `fx` straight to a
  `'Page N | Title'` measure — a text box can't take a measure binding, so this is
  what keeps a title from silently going stale the moment someone edits the page. A
  missing translation surfaces as a visible `⚠` instead of quietly falling back to
  English. Full write-up: [Dynamic Titles & Localization](docs/dynamic-titles.md).

## License

[MIT](LICENSE) — this repository (documentation, DAX/architecture write-ups) may be
reused freely with attribution. It does not include the underlying `.pbix`/PBIP
project files, which remain private.

## Author

**Fuad Gashi** — Power BI / Business Intelligence developer.
[GitHub](https://github.com/fuadgashi)
