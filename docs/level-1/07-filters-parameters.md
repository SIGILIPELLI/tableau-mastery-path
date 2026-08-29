# 07 · Filters & Parameters

Filters remove data from a view; parameters are user-adjustable values a
calculated field can react to. This module covers both, building a
threshold-adjustable view of the `Orders` table.

## 1. Dimension filters

1. Drag **Region** from the Data pane onto the **Filters** shelf. A dialog
   lists every distinct value (Central, East, West) with checkboxes — leave
   all checked to filter nothing yet, or uncheck **Central** to exclude it
   from the view entirely.
2. Once a filter is on the shelf, right-click it → **Show Filter** to add an
   interactive control (checkboxes, a dropdown, or a list, chosen from the
   card's own dropdown arrow) directly on the worksheet/dashboard so a
   viewer can change it without editing the workbook.

## 2. Measure filters

1. Drag **Sales** onto the Filters shelf. Tableau asks which aggregation to
   filter on (Sum, Average, etc. — pick **Sum**) then presents a range
   slider.
2. Set the range to, say, 500–2500. Against `Orders`, this excludes the two
   Office Supplies orders (60 and 120) and the Central Furniture Chair
   order stays in only if its Sales (800) falls in range — verify by
   listing which of the 8 orders remain: 1001 (1200), 1002 (450 — excluded,
   below 500), 1004 (800), 1005 (2200), 1006 (950), 1008 (1100) — so 1002,
   1003, 1007 are excluded, leaving 5 orders.
3. Measure filters apply **before** aggregation on the underlying rows (for
   a simple SUM), which matters once table calculations are layered on top
   (Module 8) — a measure filter and a table-calculation-based filter can
   behave differently because of when in Tableau's query pipeline each is
   applied.

## 3. Filter cards: context and order

1. Right-click a filter card on the Filters shelf → **Add to Context** turns
   it into a **context filter** (shown with a grey background), which
   Tableau applies *before* any other filters below it in the query
   pipeline — relevant once you stack multiple filters and one needs to
   restrict the pool the others operate on (e.g. filtering to Furniture
   first, then a Top-N filter on Sales within just Furniture).
2. Without context, all non-context filters apply independently at roughly
   the same stage — for this course's small dataset the distinction rarely
   changes the visible result, but it's worth knowing before Level 2's
   larger, multi-filter dashboards.

## 4. Parameters

1. In the Data pane dropdown, choose **Create Parameter...**. Name it
   `Sales Threshold`, set **Data type** to **Float**, and set a default
   **Current value** of `500`.
2. Right-click the new parameter in the Data pane (under a separate
   Parameters section, bottom of the pane) → **Show Parameter Control** to
   place an adjustable slider/input on the worksheet.
3. Create a calculated field `Above Threshold` that reacts to the
   parameter:

    ```
    SUM([Sales]) >= [Sales Threshold]
    ```

4. Drag **Order ID** to Rows and **Above Threshold** to Color. With the
   parameter at its default 500, verify by hand against `Orders`: orders
   1001, 1004, 1005, 1006, 1008 (Sales ≥ 500) should color as `TRUE`;
   1002, 1003, 1007 as `FALSE`.
5. Move the parameter control's slider to 1000 and watch the coloring update
   live — now only 1001, 1005, 1006, 1008 should read `TRUE` (950 ≥ 1000 is
   false, so 1006 flips to `FALSE`) — confirming the calculated field
   re-evaluates against the live parameter value rather than a fixed number.

## 5. Filters vs. parameters — when to use which

- A **filter** changes what data is *included* in the view — it can remove
  marks entirely.
- A **parameter** is a single stored value a calculated field, a filter's
  condition, or even an axis reference line can react to — it never removes
  data on its own; whatever formula reads it decides what to do with the
  value.
- Use a parameter when you want one control to drive *several* things at
  once (a threshold used in both a calculated field's color logic and a
  reference line), which a plain filter can't do since a filter's job is
  narrowly "include or exclude."

## Cheat sheet

| Action | How |
|---|---|
| Filter a dimension | Drag field to Filters shelf → pick values |
| Filter a measure | Drag field to Filters shelf → pick aggregation → range |
| Show a filter control | Right-click filter card → Show Filter |
| Make a context filter | Right-click filter card → Add to Context |
| Create a parameter | Data pane dropdown → Create Parameter... |
| Show a parameter control | Right-click parameter → Show Parameter Control |
| Reference a parameter in a formula | Use its name in brackets, e.g. `[Sales Threshold]` |

## Exercise

Build the `Sales Threshold` parameter and `Above Threshold` calculated field
as described. Then set the parameter to exactly 950 and, by hand from the
`Orders` table, list every Order ID that should read `TRUE` — including
correctly handling the boundary case of an order whose Sales exactly equals
950.
