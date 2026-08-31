# 04 · Custom Visualizations & Extensions Overview

Tableau's built-in chart types cover most needs, but some requirements
(a custom control, a third-party JS chart library, writing data back to a
source system) need the **Extensions API**. This module is conceptual —
verification is by hand-tracing the data that would flow to/from an
extension, against the familiar `Orders` table.

## 1. What a dashboard extension is

1. A **dashboard extension** is a sandboxed web application (HTML/JS)
   loaded into a **zone** on a Tableau dashboard, communicating with the
   workbook through the **Extensions API** (`tableau.extensions.*`
   JavaScript library) — it can read summary/underlying data from
   worksheets, read/set parameters and filters, and in newer API versions
   write data back to a source.
2. Extensions run in an iframe. Tableau Desktop/Server/Cloud must
   explicitly enable extensions (they're off by default in locked-down
   deployments) and, for network-hosted extensions, an admin allow-lists
   the domain — a governance control covered further in Module 8.

## 2. Reading data into an extension: summary vs. underlying

1. `worksheet.getSummaryDataAsync()` returns the **aggregated** data
   currently rendered in a worksheet — e.g. a bar chart of Region vs.
   SUM(Sales) returns exactly 3 rows: East 2210, West 3750, Central 920.
2. `worksheet.getUnderlyingDataAsync()` returns the **row-level** data
   feeding the view, before the aggregation shown on screen — for the same
   worksheet this returns up to 8 rows (one per `Orders` record), because
   the aggregation is a rendering step, not a data reduction.
3. Hand-check: an extension summing the `Sales` field from
   `getSummaryDataAsync()` should get 2210+3750+920 = 6880. An extension
   summing `Sales` from `getUnderlyingDataAsync()` should also get 6880
   (1200+450+60+800+2200+950+120+1100) — same total, different row count
   (3 vs. 8), which is the distinction to test for if a custom chart looks
   right but a downstream calculation is off by a factor tied to
   duplicate-counting granularity.

## 3. Setting a filter from an extension

1. `worksheet.applyFilterAsync('Region', ['East'], FilterUpdateType.Replace)`
   sets the Region filter to East only, from the extension's UI rather
   than a native Tableau filter card.
2. Verify the effect the same way as any Region filter: the dashboard's
   other worksheets (if using the same filtered field) should now total
   2210, matching Section 1's East-only figure from prior modules.

## 4. A worked mini-extension: "reset all filters" button

1. Requirement: a button in the extension zone that clears every filter on
   the dashboard back to "show all."
2. Implementation sketch: iterate `tableau.extensions.dashboardContent
   .dashboard.worksheets`, and for each worksheet call
   `worksheet.clearFilterAsync(fieldName)` for each active filter field.
3. Test by hand: before reset, with Region=East applied, total Sales
   showing is 2210. After the reset click fires, every worksheet should
   re-render at the full 6880 — if it shows something else (e.g. still
   2210, or an intermediate 3160 = 2210+950), the reset loop missed a
   worksheet or a filter field.

## 5. When to reach for an extension vs. built-in features

| Need | Built-in Tableau | Extension |
|---|---|---|
| Standard chart types, filters, actions | Yes | Not needed |
| Third-party chart library (e.g. a Sankey, custom map) | No | Yes |
| Writing values back to an external system | No (until write-back APIs) | Yes |
| Custom UI controls beyond parameters/filters | Limited | Yes |
| Must work with zero admin config | Yes | No — requires enabling extensions |

## Cheat sheet

| API call | Returns |
|---|---|
| `getSummaryDataAsync()` | Aggregated rows as rendered (e.g. 3 Region rows) |
| `getUnderlyingDataAsync()` | Row-level data (up to 8 Orders rows) |
| `applyFilterAsync(field, values, type)` | Sets a filter programmatically |
| `clearFilterAsync(field)` | Removes a filter |

## Exercise

An extension calls `getSummaryDataAsync()` on a worksheet showing
Category vs. SUM(Profit). Using Level 1's `Orders` table, hand-compute the
3 rows it should return: Furniture = 180+96-40+150 = 386, Electronics =
90+330 = 420, Office Supplies = 18+42 = 60. Confirm these sum to the
workbook's total profit, 386+420+60 = 866.
