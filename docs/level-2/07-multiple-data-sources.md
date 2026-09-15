---
description: "Working with Multiple Data Sources — Beyond a single connection's tables (Module 3), a workbook can hold several independent data sources. This module…"
---

# 07 · Working with Multiple Data Sources

Beyond a single connection's tables (Module 3), a workbook can hold several
independent **data sources**. This module covers cross-data-source
dashboards, using `Orders` alongside a second, separately-connected source,
`Targets`.

## 1. The second data source: `Targets`

Connected as its own data source (e.g. a separate Google Sheet), not joined
or related to `Orders`:

| Region | Annual Target |
|---|---|
| East | 2000 |
| West | 3500 |
| Central | 1000 |

## 2. Multiple data sources in one workbook

1. Data pane's top section lists each connected source separately (e.g.
   `Orders` and `Targets`), each with its own fields — unlike a
   relationship (Module 3), fields from two different data sources cannot
   be dragged onto the same shelf directly unless linked (Section 3).
2. Each worksheet uses exactly one **primary** data source at a time; a
   second source becomes usable on that sheet only via blending (Level 2
   Module 3, Section 4) or by placing each source's fields on separate
   sheets combined at the dashboard level.

## 3. Cross-database join (same physical connection, different DB types)

1. Tableau supports joining tables that live in **different database
   types** (e.g. an Excel `Orders` table joined to a SQL Server `Targets`
   table) directly on the Data Source page — Tableau calls this a
   **cross-database join**.
2. Verify: joining `Orders` and `Targets` on `Region` (assuming both
   accessible from one Data Source page) should reproduce each region's
   Sales next to its Target without needing a blend: East 2210 vs. target
   2000 (attainment 110.5%), West 3750 vs. 3500 (107.1%), Central 920 vs.
   1000 (92.0%). Compute: 2210/2000=1.105, 3750/3500=1.0714,
   920/1000=0.92.

## 4. Dashboard-level combination without joining

1. When true joining isn't available (e.g. `Targets` sits behind a
   different security boundary), build two separate worksheets — one per
   data source — and place both on the same dashboard.
2. Sheet A (from `Orders`): bar chart of Sales by Region (2210 / 3750 /
   920). Sheet B (from `Targets`): a reference table of Region + Annual
   Target (2000 / 3500 / 1000), positioned directly beside Sheet A so a
   viewer visually compares the two without Tableau needing to combine
   them computationally.
3. This avoids fan-out or blending pitfalls entirely, at the cost of
   losing the ability to compute a single field (like attainment %) that
   spans both sources on one sheet.

## 5. Data source parameters for switching sources

1. Build a **parameter** listing environment names ("Production",
   "Staging") and use **Data Source > Replace Data Source** patterns, or a
   parameter-driven calculated field, to let a workbook swap which
   underlying source it queries — useful for testing a dashboard against a
   staging copy of `Orders` before pointing it at production data.

## 6. Verifying a multi-source dashboard

1. With Sheet A and Sheet B on one dashboard (Section 4), manually check
   that no filter action accidentally tries to pass a field from `Orders`
   into `Targets` (e.g. `Order ID` doesn't exist in `Targets` and such an
   action would silently fail to filter anything) — confirm by clicking
   East on Sheet A and observing Sheet B's Target table is unaffected
   (expected, since it isn't linked).
2. If attainment % is required on one sheet, prefer the cross-database
   join (Section 3) specifically because it guarantees the 2210/2000,
   3750/3500, 920/1000 arithmetic happens inside one query rather than
   being approximated across two independently-rendered sheets.

## How It Actually Works

The reason fields from two separate data sources can't share a shelf
without explicit linking comes down to VizQL needing a **single connection
context** to generate one coherent query:

1. Each worksheet compiles to one query (or one query per axis in a
   dual-axis view) issued against exactly one **primary** connection.
   Dragging in a field from a second, unlinked source has nowhere valid to
   go in that generated `SELECT`/`GROUP BY` — there's no shared `FROM`
   clause connecting `Orders` and `Targets` unless they've been joined
   (Section 3) into one physical connection, or blended (aggregate-level
   combination computed after two independent queries return, per Level 2
   Module 3 Section 4).
2. A **cross-database join** (Section 3) is mechanically identical to a
   same-database join (Level 2 Module 3) at the query-planning level —
   Tableau's engine issues two separate native queries (one per source
   system, since Excel and SQL Server don't share a query dialect), pulls
   both result sets into its own in-memory/Hyper execution layer, and
   performs the join *there* rather than pushing a single federated SQL
   statement down to either source. This is exactly why cross-database
   joins tend to be slower than a same-database join: the join computation
   itself runs inside Tableau's engine, not inside whichever database is
   faster at joins.
3. **Filter actions across unlinked sources silently no-op** (Section 6.1)
   because an action's mechanism is literally "add a WHERE clause built
   from the source sheet's field values" — if the target sheet's data
   source has no field with a compatible name/role to bind that value to,
   Tableau has no clause to construct, so the action fires with an empty
   effective filter rather than an error, which is precisely why the
   verification habit (click and observe, rather than assume) matters here.

## Cheat sheet

| Situation | Approach |
|---|---|
| Two DB types, need row-level combine | Cross-database join |
| Two sources, aggregate-only combine | Blend (Level 2 Module 3) |
| Two sources, no combine needed | Separate sheets, same dashboard |
| Swap environments (prod/staging) | Parameter-driven data source |
| Verify a dashboard didn't silently fail to filter | Click a mark, confirm expected behavior |

## Exercise

Using the `Targets` table above, compute each region's shortfall or surplus
in dollars (Sales − Target): East 2210−2000=+210, West 3750−3500=+250,
Central 920−1000=−80. Confirm these three numbers sum to +380, and check
that against 6880 (total Sales) − 6500 (total Target) = 380.
