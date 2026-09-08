# 01 · Advanced Calculated Fields

Level 1 Module 6 covered aggregate ratios and row-level `IF` logic. This
module goes further: **LOD (Level of Detail) expressions**, nested logic,
and **string/date functions** — all hand-verified against the `Orders`
table from Level 1 Module 1.

## 1. Why LOD expressions exist

A normal aggregate calculation like `SUM([Sales])` computes at the
**view's** level of detail — whatever dimensions are on the shelves. LOD
expressions let a calculation compute at a level *finer, coarser, or
independent of* the view.

There are three forms:

- `{FIXED [dim] : AGG(...)}` — compute at exactly the dimension(s) named,
  ignoring whatever else is in the view.
- `{INCLUDE [dim] : AGG(...)}` — compute at the view's level of detail
  *plus* the named dimension, then re-aggregate up.
- `{EXCLUDE [dim] : AGG(...)}` — compute at the view's level of detail
  *minus* the named dimension.

## 2. FIXED: Region Total Sales

1. Create a calculated field `Region Total Sales`:

    ```
    {FIXED [Region] : SUM([Sales])}
    ```

2. Drag **Region**, **Product**, and `Region Total Sales` to Rows. Every
   product row within a region shows the *same* number — the region's
   total, not the product's — because FIXED ignores Product entirely.
3. Verify by hand: East's two products are Desk (Sales 1200) and Paper
   (Sales 60) and Bookcase (Sales 950) — wait, East has three: Desk 1200,
   Paper 60, Bookcase 950. Sum = 1200 + 60 + 950 = **2210**. Every East row
   in the view should show 2210 for `Region Total Sales`, regardless of
   which product that row is.
4. West: Monitor 450 + Laptop 2200 + Desk 1100 = **3750**. Central: Chair
   800 + Binders 120 = **920**. These three numbers (2210, 3750, 920) sum to
   6880 — the full `Orders` Sales total — confirming FIXED partitions the
   whole table without double-counting or dropping rows.

## 3. Percent of Region Total (FIXED + row-level math)

1. Create `Pct of Region Sales`:

    ```
    SUM([Sales]) / {FIXED [Region] : SUM([Sales])}
    ```

2. With Region and Product on Rows, this divides each product's sales by
   its region's fixed total. Verify East/Bookcase: 950 / 2210 ≈ **43.0%**.
   East/Desk: 1200 / 2210 ≈ **54.3%**. East/Paper: 60 / 2210 ≈ **2.7%**.
   These three percentages sum to 100.0% (43.0 + 54.3 + 2.7 = 100.0),
   confirming the FIXED denominator is correct — each region's products
   partition that region's total exactly.

## 4. INCLUDE: average order size per category, independent of region

1. Create `Avg Order Sales per Category` using INCLUDE to force
   per-order-level granularity before averaging, even if the view is
   rolled up to Category only:

    ```
    {INCLUDE [Order ID] : SUM([Sales])}
    ```

2. Then a separate field `Avg per Category` = `AVG([Avg Order Sales per
   Category])` when only Category is on Rows — this computes the average
   *order* size within a category (not the average of category totals).
3. Verify Furniture (Desk 1200, Chair 800, Bookcase 950, Desk 1100 = 4
   orders): average = (1200+800+950+1100)/4 = 4050/4 = **1012.5**. This
   matches AVG(SUM(Sales)) computed at Order ID detail, included even
   though the view itself only shows Category.

## 5. EXCLUDE: category share ignoring region

1. Create `Sales Ignoring Region`:

    ```
    {EXCLUDE [Region] : SUM([Sales])}
    ```

2. With Region and Category on Rows, every Category row across every
   Region shows that category's *global* total (Region excluded from the
   computation), not the region-specific total. Verify Furniture (any
   region row): 800 (Central) + 2150 (East: Desk 1200 + Bookcase 950) +
   1100 (West) = **4050** — matches the category-only total from Level 1.

## 6. Nested logic: a three-tier discount-eligibility field

1. Create `Discount Tier` with nested `IF`:

    ```
    IF [Profit] < 0 THEN "Review"
    ELSEIF SUM([Sales]) >= 1000 AND [Profit] / [Sales] >= 0.15 THEN "Tier A"
    ELSEIF SUM([Sales]) >= 1000 THEN "Tier B"
    ELSE "Tier C"
    END
    ```

2. Verify row-by-row (using each order's own Sales/Profit, since this is a
   row-level calc, not aggregated): Order 1001 (Sales 1200, Profit 180,
   ratio 0.15) → Sales≥1000 and ratio≥0.15 → **Tier A**. Order 1005 (Sales
   2200, Profit 330, ratio 0.15) → **Tier A**. Order 1008 (Sales 1100,
   Profit 150, ratio 0.136) → Sales≥1000 but ratio<0.15 → **Tier B**. Order
   1006 (Profit -40) → **Review**. Orders 1002, 1003, 1004, 1007 all have
   Sales < 1000 → **Tier C**.

## 7. String and date functions

1. `LEFT([Product], 4)` on `Desk` returns `"Desk"`; on `Bookcase` returns
   `"Book"` — useful for building short codes.
2. `DATENAME('month', [Order Date])` on Order 1005 (`2024-03-01`) returns
   `"March"`; `DATEPART('quarter', [Order Date])` returns `1` (Jan–Mar is
   Q1 in a standard calendar).
3. `DATEDIFF('day', [Order Date], TODAY())` — a running age-of-order field,
   useful for a "days since ordered" KPI on an operations dashboard.

## How It Actually Works

LOD expressions are the clearest place to see VizQL compile a calculation
into a genuinely **separate SQL subquery**, joined back to the main query —
not a formula evaluated inline like a row-level `IF`:

1. `{FIXED [Region] : SUM([Sales])}` compiles conceptually to a subquery
   `SELECT Region, SUM(Sales) AS RegionTotal FROM Orders GROUP BY Region`,
   computed once, independent of the view. The *outer* query — driven by
   whatever's on the shelves (Region, Product) — then **left-joins** that
   subquery's result back on Region. This is mechanically why every East
   row shows the same 2210: they're all joining against the same single
   subquery row for Region='East', regardless of which Product row they
   started from.
2. `{INCLUDE [Order ID] : SUM([Sales])}` compiles differently: the subquery
   is computed at the view's dimensions *plus* Order ID — `SELECT Category,
   Order ID, SUM(Sales) FROM Orders GROUP BY Category, Order ID` — finer
   than the outer view (which only groups by Category), so the outer query
   must re-aggregate (here, AVG) over multiple subquery rows per Category.
   This two-level re-aggregation is exactly why Section 4 needs a *second*
   calculated field (`AVG(...)`) wrapped around the INCLUDE — the INCLUDE
   subquery alone still returns one row per order, not per category.
3. `{EXCLUDE [Region] : SUM([Sales])}` runs its subquery at the view's
   detail minus Region — `SELECT Category, SUM(Sales) FROM Orders GROUP BY
   Category` — computed once per Category regardless of Region, then joined
   back to every (Region, Category) row in the outer view, which is
   mechanically why every Region's Furniture row shows the same global 4050.
4. **Evaluation order matters**: LOD subqueries are resolved before
   row-level `IF` logic and before table calculations (Level 1 Module 8) —
   this ordering (dimension filters → context filters → FIXED LOD →
   non-FIXED LOD/measure filters → table calcs) is why a FIXED LOD ignores
   an ordinary dimension filter unless that filter is a *context* filter:
   a context filter physically narrows the rows the FIXED subquery ever
   sees, while a plain filter is applied to the outer query only, after the
   FIXED subquery has already run against the unfiltered table.

## Cheat sheet

| Form | Behavior |
|---|---|
| `{FIXED [dim] : AGG}` | Computes at named dim(s) only, ignores view |
| `{INCLUDE [dim] : AGG}` | View detail + named dim, then re-aggregate |
| `{EXCLUDE [dim] : AGG}` | View detail minus named dim |
| Percent of fixed total | `SUM([X]) / {FIXED [dim]: SUM([X])}` |
| Nested IF | `IF...ELSEIF...ELSEIF...ELSE...END` |
| Month name | `DATENAME('month', [date])` |
| Quarter number | `DATEPART('quarter', [date])` |

## Exercise

Using `{FIXED [Category] : SUM([Profit])}`, compute by hand each category's
fixed profit total (Furniture, Electronics, Office Supplies) from the
`Orders` table, then build `Pct of Category Profit` = `SUM([Profit]) /
{FIXED [Category]: SUM([Profit])}` and verify the Bookcase row's
percentage is negative (since its Profit is -40) even though its category
total (Furniture, 386) is positive.
