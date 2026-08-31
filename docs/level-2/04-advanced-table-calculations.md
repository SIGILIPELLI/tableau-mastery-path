# 04 · Advanced Table Calculations

Level 1 Module 8 covered running totals and simple ranks. This module goes
deeper: **addressing/partitioning**, moving averages, and quick-table-calc
edge cases — all against `Orders` sorted by `Order Date`.

## 1. Orders sorted by date (the base view)

| Order Date | Order ID | Sales |
|---|---|---|
| 2024-01-05 | 1001 | 1200 |
| 2024-01-12 | 1002 | 450 |
| 2024-02-03 | 1003 | 60 |
| 2024-02-20 | 1004 | 800 |
| 2024-03-01 | 1005 | 2200 |
| 2024-03-15 | 1006 | 950 |
| 2024-03-22 | 1007 | 120 |
| 2024-04-02 | 1008 | 1100 |

## 2. Addressing and partitioning

1. Table calcs run along an **addressing** dimension (the direction the
   calc "moves" through) while any other dimension on the view acts as a
   **partition** (a separate reset point for each partition value).
2. With only `Order Date` on Rows, `Order Date` is the sole addressing
   field — a running total scans the full 8-row list.
3. Add `Region` to Rows *before* `Order Date`: now `Region` partitions the
   table calc — the running sum resets to 0 at the start of each Region
   group, addressing only within that Region's dates.

## 3. Running total, partitioned by Region

1. `RUNNING_SUM(SUM([Sales]))`, addressed by `Order Date`, partitioned by
   `Region`.
2. East rows in date order: 1001 (1200) → running 1200; 1003 (60) →
   running 1260; 1006 (950) → running 2210. Final East running total
   **2210** matches East's flat total from earlier modules.
3. West: 1002 (450) → 450; 1005 (2200) → 2650; 1008 (1100) → **3750**.
   Central: 1004 (800) → 800; 1007 (120) → **920**. Each partition's final
   running value matches that region's known total — the standard way to
   verify partitioning is configured correctly.

## 4. Moving average (2-order trailing window)

1. Quick Table Calculation → Moving Average, set to **2 values, trailing
   average** (current + 1 previous), addressed by `Order Date` across the
   full 8-row table (no partition).
2. Hand-compute: order 2 (1002, Sales 450) averages with order 1 (1200):
   (1200+450)/2 = **825**. Order 3 (60) averages with order 2 (450):
   (450+60)/2 = **255**. Order 4 (800) with order 3 (60): (60+800)/2 =
   **430**. Order 5 (2200) with order 4 (800): (800+2200)/2 = **1500**.
3. These hand values are what should appear on the trailing-2 moving
   average line for orders 2 through 8 — order 1 shows null/blank since
   there's no prior value to average with (or itself, depending on the
   "include current value" setting).

## 5. Percent difference from previous

1. Quick Table Calculation → Percent Difference, addressed by `Order
   Date`.
2. Order 2 vs Order 1: (450 − 1200) / 1200 = -0.625 = **-62.5%**. Order 3
   vs Order 2: (60 − 450) / 450 = -0.867 = **-86.7%**. Order 5 vs Order 4:
   (2200 − 800) / 800 = 1.75 = **+175.0%** — the Laptop order more than
   doubling the prior order's Sales.

## 6. Rank with ties (dense vs. competition)

1. Ranking `SUM(Sales)` by Product, three products tie or don't: Desk
   appears twice (1200 in East, 1100 in West) — if Product is the sole
   dimension, its `SUM(Sales)` combines both Desk orders: 1200+1100=2300.
2. Full product totals: Desk 2300, Laptop 2200, Bookcase 950, Chair 800,
   Monitor 450, Binders 120, Paper 60.
3. `RANK(SUM([Sales]))` (competition ranking, ties share rank and skip the
   next): Desk=1, Laptop=2, Bookcase=3, Chair=4, Monitor=5, Binders=6,
   Paper=7 — no ties exist in this dataset, so `RANK` and `RANK_DENSE`
   produce identical results here; note for students that they'd diverge
   only if two products tied exactly on Sales.

## 7. Table calc filters (the "hidden dimension" trap)

1. Table calculations are computed *before* a filter that's configured to
   run "at the table calc level" removes rows — using **Filter** →
   right-click a field on the Filters shelf → **Add to Context**, or use
   the table-calc-specific filter option, so a Region filter doesn't skew
   a running total that should still reflect all underlying data up to
   that point.
2. Practical check: filtering the view to hide "Central" *after* computing
   a full running total should NOT change the East/West running values —
   if it does, the filter was applied before the table calc rather than
   after, and needs to move to context or use the table calc filter.

## Cheat sheet

| Table calc | Formula concept | Verified value (example) |
|---|---|---|
| Running total (partitioned) | `RUNNING_SUM(SUM([Sales]))` | East final = 2210 |
| Moving average (trailing 2) | avg of current + 1 prior | Order 3 = 255 |
| Percent difference | `(cur − prev) / prev` | Order 5 = +175.0% |
| Rank (competition) | `RANK(SUM([Sales]))` | Desk = 1 (2300) |
| Addressing | Direction the calc scans | e.g. Order Date |
| Partitioning | Dimension that resets the calc | e.g. Region |

## Exercise

Compute the trailing-2 moving average for orders 6, 7, and 8 by hand (Sales
950, 120, 1100), then verify: order 7's moving average should be
(950+120)/2 = 535, and order 8's should be (120+1100)/2 = 610.
