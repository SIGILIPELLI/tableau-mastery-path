# 08 · Table Calculations Basics

A **table calculation** computes its result from the values already in the
view — running totals, rank, percent of total — rather than from the raw
underlying rows. This module introduces the concept with three common table
calcs against `Orders`, aggregated by month.

## 1. Setting up the base view

1. Drag **Order Date** (as a continuous Month, per Module 4's technique)
   onto Columns, and **Sales** onto Rows as SUM(Sales). You should see four
   monthly bars: January, February, March, April.
2. Compute the monthly totals by hand from `Orders` first: January =
   1200 + 450 = 1650 (orders 1001, 1002), February = 60 + 800 = 860 (orders
   1003, 1004), March = 2200 + 950 + 120 = 3270 (orders 1005, 1006, 1007),
   April = 1100 (order 1008). Confirm your worksheet's bars match these four
   numbers before continuing — and note they sum to 1650+860+3270+1100 =
   6880, the grand total SUM(Sales) across all 8 orders.

## 2. Running total

1. Right-click the **SUM(Sales)** pill on Rows → **Add Table Calculation...**.
2. Set **Calculation Type** to **Running Total**, **Summarize values
   using** to **Sum**, and **Compute Using** to **Table (Across)** (meaning:
   compute across the Month axis, left to right).
3. Verify by hand: Running Total after January = 1650. After February =
   1650 + 860 = 2510. After March = 2510 + 3270 = 5780. After April = 5780 +
   1100 = 6880 — which should equal the grand total SUM(Sales) across all 8
   orders (1200+450+60+800+2200+950+120+1100 = 6880). This double-check
   habit — a running total's last value must equal the un-running SUM — is
   the fastest way to catch a wrong **Compute Using** setting.

## 3. Percent of total

1. Right-click SUM(Sales) again → **Add Table Calculation...** → **Percent
   of Total**, Compute Using **Table (Across)**.
2. Using the monthly totals from Section 1 (Jan 1650, Feb 860, Mar 3270, Apr
   1100; grand total 6880): January's Percent of Total = 1650 / 6880 ≈
   **24.0%**. Verify each month sums to 100% across the row.
3. **Compute Using** is the setting that most often trips people up on real
   dashboards: with more than one dimension in the view (e.g. Month and
   Region both on the view), "Table (Across)" vs. "Cell" vs. a specific
   dimension changes which subtotal the percentage is relative to. Always
   sanity check that percentages in a row/column actually sum to 100% for
   the grouping you intend.

## 4. Rank

1. Start a new sheet: **Region** on Rows, **SUM(Sales)** on Columns (as a
   bar or as Text).
2. Right-click SUM(Sales) → **Add Table Calculation...** → **Rank**,
   **Compute Using** → **Table (Down)**.
3. Verify by hand using regional totals from Module 3 (East = 1200+60+950 =
   2210, West = 450+2200+1100 = 3750, Central = 800+120 = 920): West should
   Rank **1**, East **2**, Central **3**.

## 5. Table calcs vs. regular aggregation — the key distinction

A regular measure like SUM(Sales) is computed straight from the underlying
rows Tableau queries. A table calculation is computed **after** that
query returns, using only the numbers already present in the view — which
is why **Compute Using** (literally: which direction/dimension to calculate
across) is a required setting for every table calc, and why adding or
removing a dimension from the view can silently change a table calculation's
result even though the underlying data didn't change.

## Cheat sheet

| Table calc | Answers | Key setting |
|---|---|---|
| Running Total | Cumulative sum so far | Compute Using = direction to accumulate |
| Percent of Total | Share of the row/column/table | Compute Using = what "total" means |
| Rank | Position among values in view | Compute Using = direction to rank across |
| (general) | — | Always confirm Compute Using matches your intended grouping |

## Exercise

Build the Rank-by-Region view described in Section 4. Then add **Category**
as a second dimension on Rows (nested under Region) and re-check the Rank
table calculation's **Compute Using** setting — describe in one or two
sentences how the ranking's meaning changes once Category is added, and
what **Compute Using** value you'd need to keep ranking regions against
each other rather than ranking region-category pairs against each other.
