# DAX Measure Catalog

Formula logic is reproduced exactly as authored; only identifiers are translated to
English. Every current/prior-year/growth-index variant composes the
[shared functions](dax-functions.md) rather than being hand-written per measure.

## 1. Purchase value & quantity

```dax
Purchase Value =
    SUMX('Goods Receipt Lines', 'Goods Receipt Lines'[Quantity] * 'Goods Receipt Lines'[SupplyBasePrice])

Purchase Value, Current Year YTD =
    YearToDate([Purchase Value])

Purchase Value, Prior Year =
    PriorYear([Purchase Value])

Purchase Value, Prior Year YTD =
    YearToDate([Purchase Value, Prior Year])

Purchase Value, YoY Growth Index =
    GrowthIndex([Purchase Value, Current Year YTD], [Purchase Value, Prior Year])

Purchase Quantity =
    SUM('Goods Receipt Lines'[Quantity])

Purchase Quantity, Current Year YTD =
    YearToDate([Purchase Quantity])

Purchase Quantity, Prior Year =
    PriorYear([Purchase Quantity])

Purchase Quantity, Prior Year YTD =
    YearToDate([Purchase Quantity, Prior Year])

Purchase Quantity, YoY Growth Index =
    GrowthIndex([Purchase Quantity, Current Year YTD], [Purchase Quantity, Prior Year])
```

## 2. Purchase transaction volume

Tracked as first-class KPIs alongside value/quantity — see the
[design note](data-model.md#design-note-transactions-not-just-totals) on why
transaction count matters independently of purchase value.

```dax
Purchase Invoice Count =
    DISTINCTCOUNT('Goods Receipt Lines'[InvoiceNumber])

Purchase Invoice Count, Current Year YTD =
    YearToDate([Purchase Invoice Count])

Purchase Invoice Count, Prior Year =
    PriorYear([Purchase Invoice Count])

Purchase Invoice Count, Prior Year YTD =
    YearToDate([Purchase Invoice Count, Prior Year])

Purchase Invoice Count, YoY Growth Index =
    GrowthIndex([Purchase Invoice Count, Current Year YTD], [Purchase Invoice Count, Prior Year])

Products Purchased =
    DISTINCTCOUNT('Goods Receipt Lines'[ProductId])

Products Purchased, Current Year YTD =
    YearToDate([Products Purchased])

Products Purchased, Prior Year =
    PriorYear([Products Purchased])

Products Purchased, YoY Growth Index =
    GrowthIndex([Products Purchased, Current Year YTD], [Products Purchased, Prior Year])
```

## 3. Brand & product share

```dax
Purchase Share by Brand, Current Year =
    DIVIDE(
        [Purchase Value, Current Year YTD],
        CALCULATE([Purchase Value, Current Year YTD], REMOVEFILTERS(Brands)),
        0
    )
```
