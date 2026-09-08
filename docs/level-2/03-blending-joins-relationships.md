# 03 · Data Blending vs Joins vs Relationships

Level 1 used one flat `Orders` table. Real projects split data across
tables — this module covers the three ways Tableau combines them, using a
second table, `Reps` (sales reps by Region), alongside `Orders`.

## 1. The two tables

`Orders` (from Level 1 Module 1, 8 rows) and a new lookup table `Reps`:

| Region | Rep Name | Rep Quota |
|---|---|---|
| East | Alice | 2000 |
| West | Bob | 3000 |
| Central | Carol | 1000 |

Both share the `Region` field — the join/relationship key throughout this
module.

## 2. Joins (physical, single-table result)

1. On the Data Source page, drag both tables onto the canvas; Tableau
   defaults to a **relationship** (Section 3) — click the join icon between
   them to instead configure a physical **join**: Inner, Left, Right, or
   Full Outer, matched on `Region = Region`.
2. **Inner join**: keeps only rows where Region exists in both tables.
   Since all three Regions (East, West, Central) exist in both `Orders` and
   `Reps`, an inner join here returns all 8 `Orders` rows, each duplicated
   with its rep's name and quota attached — no rows lost.
3. To see a join actually drop or duplicate rows, imagine a 4th `Reps` row
   for "South" with no matching `Orders` rows: an inner join excludes
   South entirely (0 orders), while a **left join** from `Orders` still
   excludes it too (South never appears as an Order region); a left join
   from `Reps` would keep South as one row with null Order fields.
4. **Join culprit — duplicated rows**: if `Reps` instead had *two* rows for
   East (e.g. Alice and a second East rep), joining would duplicate every
   East `Orders` row once per matching `Reps` row, silently inflating
   `SUM(Sales)` for East. Always verify total Sales stays 6880 after any
   join — this is the standard sanity check.

## 3. Relationships (default, noeditable double line)

1. A **relationship** is the modern default when dragging a second table
   onto the canvas — it does *not* pre-join the tables into one row set.
   Instead, Tableau queries each table at its **native level of detail**
   and combines results only when a sheet actually uses fields from both.
2. Because no physical join happens up front, a relationship avoids the
   row-duplication risk in Section 2.4: `SUM([Sales])` from `Orders`
   stays 6880 regardless of how many `Reps` rows share a Region, since
   Sales is aggregated within `Orders` before combining.
3. Verify: build a view with `Region`, `SUM(Sales)`, and `Rep Name`. East
   shows Sales 2210 with Rep "Alice" — the relationship correctly resolves
   the many-Orders-to-one-Rep link without duplicating the 2210 total.

## 4. Data blending (separate data sources)

1. Blending applies when `Orders` and `Reps` are two **separate published
   data sources** (not two tables in one connection) — e.g. `Orders` lives
   in an Excel file and `Reps` in a separate Google Sheet.
2. Blending works at the **aggregate** level: the secondary source's data
   is aggregated to match the primary sheet's granularity *before*
   combining, using a common linking field (Region, marked with a small
   chain-link icon in the field list).
3. Practical consequence: blending cannot push filters from the secondary
   source back onto the primary source's row-level detail — it's a
   one-directional, aggregate-only combination, which is why Tableau
   recommends relationships/joins whenever both tables are reachable from
   one connection.

## 5. Choosing among the three

| Situation | Use |
|---|---|
| Tables in the same connection, need row-level combined detail | Join |
| Tables in the same connection, different levels of detail (as here) | Relationship |
| Tables from genuinely separate data sources/connections | Blend |
| Risk of fan-out row duplication from a one-to-many key | Prefer relationship over join |

## 6. Verifying with hand arithmetic

1. Build `SUM(Sales)` by Region using the relationship from Section 3:
   East 2210, West 3750, Central 920 — identical to Level 1's single-table
   numbers, confirming the relationship introduced no duplication.
2. Add `Rep Quota` (from `Reps`) alongside: East 2210 vs quota 2000 (110%
   attainment), West 3750 vs 3000 (125%), Central 920 vs 1000 (92%).
   Compute attainment by hand: 2210/2000=1.10, 3750/3000=1.25,
   920/1000=0.92 — these become the `Sales / [Rep Quota]` calculated
   field, verified against the manual division.

## How It Actually Works

The fan-out bug in Section 2.4 and the reason relationships avoid it both
come down to **when aggregation happens relative to the row combination**:

1. A **physical join** happens at the row level, before any `GROUP BY` — a
   join produces one combined intermediate row set first (e.g. `Orders JOIN
   Reps ON Region`), and *then* VizQL's `SUM(Sales)` aggregates over
   *however many rows that join produced*. With a second East rep (Dana),
   the join produces 4 East rows instead of 3 for the combined result (each
   of East's three Orders rows duplicated once per matching Reps row: 3
   Orders × 2 Reps rows = 6 combined East rows total, each carrying its
   original Sales value), so `SUM(Sales)` for East sums 1200+1200+60+60
   +950+950 = 4420 — double-counting each Orders row once per matching Reps
   row, not "duplicating the sum" as a separate step but literally
   re-summing physically duplicated rows.
2. A **relationship** avoids this because its generated query never
   physically joins raw rows across granularities — Tableau computes
   `SUM(Sales)` within `Orders` at `Orders`' own native grain first (a
   `GROUP BY Region` subquery returning one row per Region, still 2210 for
   East), and only *then* performs a row-preserving lookup against `Reps`
   for the Rep Name/Quota columns — conceptually a `LEFT JOIN` of two
   already-aggregated result sets rather than a join of raw fact rows, so
   there's no intermediate row list a duplicate `Reps` key can inflate.
3. **Blending** (Section 4) works at a third, coarser stage still: the
   secondary source's own query runs completely independently and is
   pre-aggregated to the *primary* sheet's exact dimensions before any
   combination — so a duplicate Rep row in a blended secondary source can't
   even affect the primary's Sales total, because the primary's aggregate
   query never references the secondary source's rows at all; only the
   final displayed numbers are combined, client-side, after both queries
   have independently returned.

## Cheat sheet

| Concept | Key trait |
|---|---|
| Join | Physical row combination; risk of fan-out duplication |
| Relationship | Default; aggregates each table at native detail first |
| Blend | For separate data sources; aggregate-only, one-directional |
| Fan-out risk | One-to-many join key duplicates the "one" side's measures |
| Sanity check | Grand total (6880) should survive any join/relationship |

## Exercise

Add a 4th `Reps` row: Region "East", Rep Name "Dana", Rep Quota 500 (so
East now has two reps). Using an **inner join** (not a relationship),
explain by hand why `SUM(Sales)` for East would incorrectly show 4420
(double 2210) instead of 2210, and why switching to a relationship fixes it.
