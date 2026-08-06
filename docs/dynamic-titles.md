# Dynamic Titles & Localization

Every piece of user-facing text in this report — page titles, subtitles,
navigator tabs, KPI card headers, slicer captions — is a DAX measure lookup, not a
hardcoded string. This page is the full reference for that system: the three tables
behind it, the naming/key scheme, a worked example for every label family, and how a
label actually reaches the screen.

## Architecture

```mermaid
flowchart LR
    Sec[("Security\nUser · Culture · Role")] -->|SELECTEDVALUE| UC["[User Culture] measure"]
    UC --> DL["Dynamic Labels\none measure per label"]
    Lab[("Labels\nPageKey · Culture · LabelType · Label")] -->|CALCULATE + SELECTEDVALUE| DL
    DL -->|fx binding| Vis["Report visual\n(hidden action button, KPI card, slicer)"]
```

Three tables, three jobs:

| Table | Job |
|---|---|
| `Security` | Resolves *who* is looking and *which* culture they should see. |
| `Labels` | Holds *what* every label says, in every supported culture. Plain data — no DAX. |
| `Dynamic Labels` | Holds *the lookup* — one measure per label, each reading `Labels` filtered to its own key and the current culture. |

## The key scheme

`Labels[PageKey]` is a plain integer, but it's not arbitrary — it encodes both *which
page* and *which role* through numeric bands, so the whole label set for the report
sorts and scans predictably instead of needing a lookup table of its own:

| Band | Role | Formula |
|---|---|---|
| 1–2 | Page title | `PageKey = N` |
| 201–202 | Page subtitle | `PageKey = 200 + N` |
| 401–402 | Page navigator-tab label | `PageKey = 400 + N` |
| 1200s | Visual title | one `PageKey` per visual, allocated in build order |
| 1400s | Visual subtitle | one `PageKey` per visual, allocated in build order |

*N* is the page number in report order (see [Report Tour](report-pages.md)):

| N | Page |
|---|---|
| 1 | Supply |
| 2 | Purchases |

## `Labels` — representative rows

```
PageKey | PageName  | Culture | LabelType        | Label
--------|-----------|---------|-------------------|---------------------------------------
      1 | Supply    | en-US   | Title             | Supply
      1 | Supply    | sq-AL   | Title             | Furnizim
    201 | Supply    | en-US   | Subtitle          | Incoming goods receipts over time
    201 | Supply    | sq-AL   | Subtitle          | Furnizimet hyrëse me kalimin e kohës
    401 | Supply    | en-US   | Navigator Title   | Supply
      2 | Purchases | en-US   | Title             | Purchases
      2 | Purchases | sq-AL   | Title             | Blerjet
    402 | Purchases | en-US   | Navigator Title   | Purchases
   1201 | TrendChart      | en-US | VisualTitle   | Purchase Value & Quantity Trend
   1201 | TrendChart      | sq-AL | VisualTitle   | Trendi i Vlerës dhe Sasisë së Blerjes
   1203 | TransactionTable | en-US | VisualTitle  | Transaction Detail
   1220 | BrandProductPivot | en-US | VisualTitle | Purchase by Brand & Product
   1221 | ShareByBrandDonut | en-US | VisualTitle | Purchase Share by Brand
   1421 | ShareByBrandDonut | en-US | VisualSubtitle | Share of total purchase value, current year
```

A Power Query guard on the `Labels` partition errors the refresh on a duplicate
`PageKey + Culture + LabelType` combination — a copy-pasted row that wasn't
re-keyed fails loudly at refresh time, not silently at render time.

## `Dynamic Labels` — the lookup pattern

Every measure in this table follows the same three-line shape: resolve the culture,
then `CALCULATE` a `SELECTEDVALUE` against `Labels` filtered to this label's exact
key and that culture.

```dax
'Page 1 | Title' =
    VAR _culture = [User Culture]
    RETURN
        CALCULATE (
            SELECTEDVALUE ( Labels[Label], "⚠" ),
            Labels[LabelType] = "Title",
            Labels[PageKey] = 1,
            Labels[Culture] = _culture
        )

'Page 1 | Subtitle' =
    VAR _culture = [User Culture]
    RETURN
        CALCULATE (
            SELECTEDVALUE ( Labels[Label], "⚠" ),
            Labels[LabelType] = "Subtitle",
            Labels[PageKey] = 201,
            Labels[Culture] = _culture
        )

'Page 2 | Navigator Title' =
    VAR _culture = [User Culture]
    RETURN
        CALCULATE (
            SELECTEDVALUE ( Labels[Label], "⚠" ),
            Labels[LabelType] = "Navigator Title",
            Labels[PageKey] = 402,
            Labels[Culture] = _culture
        )

'Trend Chart | VisualTitle' =
    VAR _culture = [User Culture]
    RETURN
        CALCULATE (
            SELECTEDVALUE ( Labels[Label], "⚠" ),
            Labels[LabelType] = "VisualTitle",
            Labels[PageKey] = 1201,
            Labels[Culture] = _culture
        )
```

The `"⚠"` second argument to `SELECTEDVALUE` is the fallback when no row matches —
deliberately a visible warning glyph, not a quiet default to English. A missing
translation shows up as `⚠` on the page during review, instead of shipping a report
that silently reads wrong for one culture and looks fine for the other.

### The page-navigator exception

Each page's left-hand navigator tab needs its own label, but that label is *the same
text* as the page's own Navigator Title — so rather than adding a second `Labels` row
per page, `'PageNavN | Title'` re-reads the existing `400 + N` row directly:

```dax
'PageNav1 | Title' =
    VAR _culture = [User Culture]
    RETURN
        CALCULATE (
            SELECTEDVALUE ( Labels[Label], "⚠" ),
            Labels[LabelType] = "Navigator Title",
            Labels[PageKey] = 401,
            Labels[Culture] = _culture
        )
```

One `Labels` row, two consumers (`'Page 1 | Navigator Title'` and
`'PageNav1 | Title'`) — the nav bar and the in-page navigator always agree because
they're reading the same source row, never two copies that can drift apart.

## Binding a label to a visual

A measure existing isn't what makes a title dynamic — the visual has to actually be
bound to it. Every page title in this report is a **hidden action-button visual**:
icon, outline, text, and fill are all set to `show: false`, and only the
`visualContainerObjects.title`/`subTitle` properties are shown, each bound via `fx`
to the matching measure:

```json
"title": [{
  "properties": {
    "show": { "expr": { "Literal": { "Value": "true" } } },
    "text": {
      "expr": {
        "Measure": {
          "Expression": { "SourceRef": { "Entity": "Dynamic Labels" } },
          "Property": "Page 1 | Title"
        }
      }
    }
  }
}]
```

Never a text box — a text box's content is a literal string with no `fx` option, so
a hardcoded title has no path back to `Labels` and quietly stops translating the
first time someone edits the page. The transaction table's title and the
purchase-share donut's title/subtitle bind the same way, through their own
`VisualTitle`/`VisualSubtitle` measures.

## Adding a language

Because every label is a data row, not a code branch, supporting a third culture is
a data change, not a model change:

1. Add one `Security[Culture]` value for the pilot user(s).
2. Add one `Labels` row per existing label, in the new culture, same `PageKey` +
   `LabelType`.
3. Nothing in `Dynamic Labels` changes — every measure already filters on whatever
   `[User Culture]` resolves to.

No new measures, no new `SWITCH` branches, no touching the report canvas. The
`Labels` table is the entire surface area of a localization change.
