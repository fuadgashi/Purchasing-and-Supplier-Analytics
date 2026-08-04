# DAX Functions Used in This Module

Purchase & Supply draws on 3 functions from the solution's shared DAX
time-intelligence library — the same reusable engine used across every module. Only
the functions this module actually calls are documented below.

```dax
/// Year-to-date: evaluates the measure for the current calendar year, dates up to
/// and including today. KEEPFILTERS preserves any existing date filters.
function YearToDate =
    (Measure: anyref expr) =>
        CALCULATE(
            Measure,
            KEEPFILTERS(Calendar[Year] = YEAR(TODAY())),
            KEEPFILTERS(Calendar[Date] <= TODAY())
        )

/// Prior year: shifts the measure back one year using DATEADD and restricts to the
/// previous calendar year. Used for year-over-year comparisons.
function PriorYear =
    (Measure: anyref expr) =>
        CALCULATE(
            Measure,
            DATEADD(Calendar[Date], -1, YEAR),
            Calendar[Year] = YEAR(TODAY()) - 1
        )

/// Percentage difference (CurrentValue - PriorValue) / PriorValue for normal
/// positive-base measures. Guards: blank when both blank or current-blank on a
/// future date; returns 1 when current exists but prior was blank on a past date, or
/// when prior is negative.
function GrowthIndex =
    (CurrentValue: decimal val, PriorValue: decimal val) =>
        VAR _IsCurrentBlank = ISBLANK(CurrentValue)
        VAR _IsPriorBlank   = ISBLANK(PriorValue)
        VAR _IsFutureDate   = MAX(Calendar[Date]) > TODAY()
        VAR _IsPastDate     = MAX(Calendar[Date]) < TODAY()
        RETURN
            SWITCH(
                TRUE(),
                _IsCurrentBlank && _IsPriorBlank, BLANK(),
                _IsCurrentBlank && _IsFutureDate && NOT (_IsPriorBlank), BLANK(),
                NOT (_IsCurrentBlank) && _IsPriorBlank && _IsPastDate, 1,
                NOT (_IsCurrentBlank) && PriorValue < 0, 1,
                DIVIDE(CurrentValue - PriorValue, PriorValue)
            )
```

These 3 are drawn from a larger 23-function library covering the full solution's
time-intelligence needs — see the [Sales Analytics](https://github.com/fuadgashi/Sales-Analytics)
repository for the complete set.
