# 06 · Integrating Tableau with Databases at Scale

This module covers connecting Tableau to production-scale database
patterns — joins across large tables, custom SQL, and cross-database
joins — using `Orders` split across two related tables to keep every
number hand-checkable.

## 1. Splitting `Orders` into a star-ish schema

For this module, `Orders` is normalized into two tables:

**`OrderFacts`**

| Order ID | Region Code | Sales | Profit |
|---|---|---|---|
| 1001 | R-E | 1200 | 180 |
| 1002 | R-W | 450 | 90 |
| 1003 | R-E | 60 | 18 |
| 1004 | R-C | 800 | 96 |
| 1005 | R-W | 2200 | 330 |
| 1006 | R-E | 950 | -40 |
| 1007 | R-C | 120 | 42 |
| 1008 | R-W | 1100 | 150 |

**`RegionDim`**

| Region Code | Region Name |
|---|---|
| R-E | East |
| R-W | West |
| R-C | Central |

## 2. Joining in Tableau vs. pushing the join to the database

1. A Tableau **join** (Data pane, drag both tables in, join on `Region
   Code`) reconstructs the same 8-row table as Level 1's `Orders`, with
   `Region Name` in place of the raw code — verify: filtering the joined
   result to Region Name = East should return rows 1001, 1003, 1006,
   summing Sales to 1200+60+950 = 2210, matching every prior module.
2. For large fact tables (millions of rows) joined to small dimension
   tables (a handful of rows, like `RegionDim`'s 3), pushing the join to
   the database via **custom SQL** —
   `SELECT f.*, d.[Region Name] FROM OrderFacts f JOIN RegionDim d ON
   f.[Region Code] = d.[Region Code]` — lets the database's own optimizer
   and indexes handle the join, which typically outperforms Tableau
   re-joining a live connection's raw tables row by row on every query.
3. Trade-off: custom SQL is treated by Tableau as a single opaque table —
   individual columns from `OrderFacts`/`RegionDim` are no longer available
   separately for join optimization, and Tableau can't push filters down
   into a custom SQL query as efficiently as it can into an ordinary table
   reference. Use a native join first; reach for custom SQL only when the
   database-side join is measurably faster (Module 5's Performance
   Recording) or a query needs SQL Tableau's join UI can't express.

## 3. Join type verification

1. An **inner join** on `Region Code` keeps only matching rows — here,
   every `OrderFacts` row has a matching `RegionDim` row, so an inner join
   returns all 8 rows.
2. Confirm what happens with an unmatched row: add a 9th `OrderFacts` row
   with `Region Code = "R-S"` (South — not in `RegionDim`). An inner join
   drops this row entirely (9 → 8 visible rows, Sales total unchanged at
   6880); a **left join** (OrderFacts as the left table) keeps it with
   `Region Name` = null, so a Sales total by Region Name would still show
   6880 across the 3 known regions, plus the new row's Sales appearing
   under a null/blank Region Name group — the grand total across *all*
   rows (6880 + new row's Sales) diverges from the by-Region-name subtotal
   unless the null group is included.

## 4. Cross-database join

1. A **cross-database join** joins tables from two different connections
   (e.g. `OrderFacts` in a SQL Server database, `RegionDim` in an Excel
   file) — Tableau performs this join itself (it can't push it to either
   source's engine, since no single engine sees both tables), so it's
   inherently a Tableau-side (not push-down) join, generally read via an
   extract for tables above trivial size.
2. For `OrderFacts`/`RegionDim`, a cross-database join produces the exact
   same 8-row, 2210/3750/920-by-region result as the same-database join in
   Section 2 — cross-database joins change *where* the join executes, not
   the logical result, provided the join key (`Region Code`) matches
   exactly (case, whitespace) across both sources, per Module 3's cleaning
   lessons.

## 5. Connection pooling and query load at scale

1. Multiple Tableau Server users viewing a live-connected dashboard each
   generate separate queries against the source database — for `Orders`'
   scale this is negligible, but at production scale this is why Tableau
   Server's Data Server component (Module 2) caches results and why
   extracts (refreshed on a schedule, e.g. nightly) are typically preferred
   for high-concurrency published dashboards over many simultaneous live
   connections hitting the same OLTP database.

## Cheat sheet

| Pattern | Where the join runs | Best for |
|---|---|---|
| Native join (Data pane) | Tableau (live) or extract build | Small-to-medium dims joined to a fact |
| Custom SQL | Database | Complex logic, DB-side optimization |
| Cross-database join | Tableau only | Tables live in different systems |
| Left join with unmatched key | Tableau/DB per join location | Preserving orphan rows (nulls surface) |

## Exercise

`OrderFacts` gains Order ID 1009, Region Code `"R-S"`, Sales 300, Profit
45 — with no matching row in `RegionDim`. Using a **left join**
(OrderFacts left), hand-compute the grand total Sales across all rows
(6880 + 300 = **7180**) and explain why a view broken out **by Region
Name** would still show only East/West/Central/(Null) as the four groups,
with the Null group holding exactly the 300 from row 1009.
