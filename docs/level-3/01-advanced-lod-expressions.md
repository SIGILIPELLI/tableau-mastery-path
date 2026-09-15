---
description: "Advanced LOD Expressions — Level 2 Module 1 introduced FIXED/INCLUDE/EXCLUDE individually. This module covers nested LOD expressions and common pitfalls…"
---

# 01 · Advanced LOD Expressions

Level 2 Module 1 introduced FIXED/INCLUDE/EXCLUDE individually. This
module covers **nested LOD expressions** and common pitfalls, still against
the `Orders` table (8 rows, Level 1 Module 1).

## 1. Recap of totals used throughout

Region: East 2210, West 3750, Central 920. Category: Furniture 4050,
Electronics 2650, Office Supplies 180. Grand total: 6880.

## 2. Nested LOD: region rank among categories within region

1. Goal: for each order, show what fraction its Category is of its
   Region's total — combining an EXCLUDE-style share with a FIXED
   denominator computed per Region *and* Category simultaneously.
2. `Category Sales in Region` = `{FIXED [Region], [Category] :
   SUM([Sales])}`. East/Furniture (Desk 1200 + Bookcase 950 = 2150),
   East/Office Supplies (Paper 60), West/Furniture (Desk 1100),
   West/Electronics (Monitor 450 + Laptop 2200 = 2650), Central/Furniture
   (Chair 800), Central/Office Supplies (Binders 120).
3. `Pct of Region for Category` = `SUM([Sales]) / {FIXED [Region] :
   SUM([Sales])}` applied to the `Category Sales in Region` figures:
   East/Furniture 2150/2210 = 97.3%; East/Office Supplies 60/2210 = 2.7%
   (these two sum to 100%, since East has no Electronics orders).
   West/Furniture 1100/3750 = 29.3%; West/Electronics 2650/3750 = 70.7%
   (sum 100%). Central/Furniture 800/920 = 87.0%; Central/Office Supplies
   120/920 = 13.0% (sum 100%).

## 3. LOD inside another LOD (double nesting)

1. `Avg Category Total Across Regions` = `{FIXED [Category] : AVG({FIXED
   [Region],[Category] : SUM([Sales])})}` — the inner FIXED computes each
   Region/Category cell (Section 2.2), the outer AVG collapses those cells
   to one number per Category.
2. Furniture has three Region/Category cells: East 2150, West 1100,
   Central 800. Average = (2150+1100+800)/3 = 4050/3 = **1350**. Note this
   differs from Furniture's flat total (4050) — the nested LOD answers "on
   average, how much Furniture sales per region," not "total Furniture
   sales."
3. Electronics has one cell only (West 2650, since East/Central have no
   Electronics orders) — its average equals 2650 itself, since AVG of a
   single value is that value.

## 4. The "LOD ignores filters" pitfall

1. By default, a FIXED LOD is computed **before** dimension filters are
   applied (though after context filters/data source filters) — filtering
   the view to `Region = East` does **not** change `{FIXED [Category] :
   SUM([Sales])}`, which still returns the full 4050/2650/180 totals, not
   East-only figures.
2. Verify: with a Region filter set to East only, a table showing Category
   and `{FIXED [Category]: SUM([Sales])}` should still display 4050 next
   to Furniture — not 2150 (East's Furniture-only total) — because the
   filter isn't in the FIXED expression's dimension list.
3. Fix: to make the LOD respect the filter, either add `[Region]` into the
   FIXED list, or convert the filter to a **context filter** — a context
   filter (Level 2 Module 8) *does* apply before FIXED LODs.

## 5. LOD vs. table calculation — when each applies

| Need | Use |
|---|---|
| Aggregate independent of view, respects/ignores filters precisely | LOD |
| Result depends on the mark's position/order in the rendered table | Table calc |
| Needs to work identically before any dimension is dropped on shelf | LOD |
| Running total, rank, moving average | Table calc |

## How It Actually Works

Nested LODs compile to genuinely **nested subqueries**, evaluated inside
out — and the filter-ordering pitfall in Section 4 is a direct consequence
of exactly *when* in VizQL's pipeline a FIXED subquery gets to run:

1. `{FIXED [Category] : AVG({FIXED [Region],[Category] : SUM([Sales])})}`
   compiles to something like: inner subquery `SELECT Region, Category,
   SUM(Sales) AS cell FROM Orders GROUP BY Region, Category` runs first,
   producing the 6 region×category cells from Section 2.2; the outer FIXED
   then runs a *second* aggregation over that inner result — `SELECT
   Category, AVG(cell) FROM (<inner>) GROUP BY Category` — which is exactly
   why Furniture's answer (1350) is the average of three already-summed
   cells (2150, 1100, 800), not a re-scan of the original 8 rows with a
   different grouping.
2. **Pipeline ordering explains the filter pitfall precisely**: Tableau's
   documented evaluation order is roughly (1) extract/data-source filters,
   (2) context filters, (3) FIXED LOD expressions, (4) ordinary
   dimension/measure filters and non-FIXED (INCLUDE/EXCLUDE) LODs, (5)
   table calculations. A plain Region filter sits at stage 4 — *after*
   FIXED has already run at stage 3 — so by the time the filter would
   remove non-East rows, the FIXED subquery's result (4050/2650/180) has
   already been computed from the *unfiltered* table and is just being
   joined back to the (now filtered) outer view. Promoting the filter to a
   context filter moves it to stage 2, ahead of FIXED, so the inner
   subquery itself only ever sees the East-only rows.
3. This staged model is also why INCLUDE/EXCLUDE (stage 4) behave
   differently from FIXED (stage 3) with respect to filters: an EXCLUDE
   LOD, sitting at the same stage as an ordinary filter, generally *does*
   respect non-context dimension filters, since its subquery is evaluated
   from whatever rows survive up to that point in the pipeline — the
   FIXED-specific "ignores filters" behavior in Section 4 doesn't
   automatically generalize to every LOD keyword, a distinction worth
   testing explicitly with hand arithmetic rather than assuming.

## Cheat sheet

| Expression | Result for this dataset |
|---|---|
| `{FIXED [Region],[Category]: SUM(Sales)}` | 6 region×category cells |
| `{FIXED [Category]: AVG({FIXED [Region],[Category]: SUM(Sales)})}` | Furniture=1350 |
| FIXED + non-context filter | Ignores the filter |
| FIXED + context filter | Respects the filter |

## Exercise

Compute `Avg Category Total Across Regions` for Office Supplies by hand:
its two cells are East 60 and Central 120 (West has none). Average =
(60+120)/2 = 90. Confirm this differs from Office Supplies' flat total
(180) for the same reason Furniture's did in Section 3.
