# 10 · Project — Build a Sales Performance Dashboard

This capstone combines every Level 1 module into one deliverable: a
multi-sheet, interactive Sales Performance Dashboard for Northwind Retail,
built against the `Orders` table (with the State column from Module 9).

## Dataset recap

**Table: `Orders`**

| Order ID | Order Date | Region | State | Category | Product | Sales | Quantity | Profit |
|---|---|---|---|---|---|---|---|---|
| 1001 | 2024-01-05 | East | New York | Furniture | Desk | 1200 | 2 | 180 |
| 1002 | 2024-01-12 | West | California | Electronics | Monitor | 450 | 3 | 90 |
| 1003 | 2024-02-03 | East | New York | Office Supplies | Paper | 60 | 10 | 18 |
| 1004 | 2024-02-20 | Central | Texas | Furniture | Chair | 800 | 4 | 96 |
| 1005 | 2024-03-01 | West | California | Electronics | Laptop | 2200 | 2 | 330 |
| 1006 | 2024-03-15 | East | Massachusetts | Furniture | Bookcase | 950 | 1 | -40 |
| 1007 | 2024-03-22 | Central | Texas | Office Supplies | Binders | 120 | 8 | 42 |
| 1008 | 2024-04-02 | West | Washington | Furniture | Desk | 1100 | 1 | 150 |

## Step 1 — Build four worksheets

1. **"Sales by Region"** (Module 3/4 technique) — Region on Columns,
   SUM(Sales) on Rows, bar chart, sorted descending. Expected totals: East
   2210, West 3750, Central 920.
2. **"Sales Over Time"** (Module 4) — continuous Month of Order Date on
   Columns, SUM(Sales) on Rows, line chart. Expected monthly totals: Jan
   1650, Feb 860, Mar 3270, Apr 1100.
3. **"Profit Ratio by Category"** (Module 6) — Category on Rows, the
   `Profit Ratio` calculated field (`SUM([Profit]) / SUM([Sales])`) on
   Columns as a bar, formatted as a percentage. Expected: Furniture =
   (180+96-40+150)/(1200+800+950+1100) = 386/4050 ≈ 9.53%; Electronics =
   (90+330)/(450+2200) = 420/2650 ≈ 15.85%; Office Supplies =
   (18+42)/(60+120) = 60/180 ≈ 33.33%.
4. **"Sales by State"** (Module 9) — filled map, State on Detail,
   SUM(Sales) on Color. Expected darkest state: California (2650).

## Step 2 — Assemble the dashboard

1. Create a new dashboard sized **Desktop Browser**. Add a **Text** title
   object: "Northwind Retail — Sales Performance Dashboard".
2. Arrange the four worksheets in a 2×2 tiled grid: "Sales by Region" (top
   left), "Sales Over Time" (top right), "Profit Ratio by Category" (bottom
   left), "Sales by State" (bottom right).
3. Add a global **Region** filter (Module 5, Section 5): right-click the
   Region filter card on any one sheet → **Apply to Worksheets > All Using
   This Data Source**, then show its control on the dashboard.

## Step 3 — Wire interactivity

1. **Dashboard > Actions... > Add Action > Filter**: Source = "Sales by
   Region", Target = "Sales Over Time" and "Profit Ratio by Category", Run
   on **Select**. Clicking a region bar should now filter both the trend
   line and the category ratio chart to that region alone.
2. **Dashboard > Actions... > Add Action > Highlight**: Source = "Sales by
   State", Target = "Sales by Region", Run on **Hover**. Hovering a state on
   the map should highlight its region's bar without filtering anything out.

## Step 4 — Verification pass (do this by hand, not just visually)

Confirm each of the following against the dashboard you built:

1. With no filters applied, "Sales by Region" shows West as the tallest bar
   at 3750.
2. Clicking the East bar filters "Sales Over Time" so only January (from
   order 1001, since order 1003/1006 are February/March — recompute:
   East's orders are 1001 (Jan), 1003 (Feb), 1006 (Mar), so the filtered
   line should show three points — January 1200, February 60, March 950)
   and "Profit Ratio by Category" to only East's categories (Furniture and
   Office Supplies, not Electronics, since East has no Electronics orders).
3. "Sales by State" shows California as the darkest polygon (2650), and
   hovering it highlights the West bar in "Sales by Region" without
   removing East or Central from view.
4. Deselecting (clicking the East bar again) returns all three linked
   sheets to their unfiltered totals from Steps 1 and Section above.

## How It Actually Works

This capstone dashboard is a good place to see the full query pipeline from
earlier modules working together on one canvas: each of the four
worksheets still issues its own independent `GROUP BY` query (Module 1's
lesson), the global Region filter (Module 7) prepends a shared `WHERE`
clause to all of them from one connection, and the filter/highlight actions
(Module 5) inject or remove additional per-click `WHERE` clauses and
opacity states on top of that:

1. When the global Region filter is set to all regions, "Sales by Region"
   runs `SELECT Region, SUM(Sales) FROM Orders GROUP BY Region`, returning
   East 2210, West 3750, Central 920 (800+120) — matching Step 1's
   hand-computed totals and confirming West is tallest.
2. Clicking the East bar adds a *second*, action-driven `WHERE Region =
   'East'` on top of whatever the global filter already allows, re-running
   "Sales Over Time"'s query as `SELECT Month, SUM(Sales) FROM Orders WHERE
   Region='East' GROUP BY Month` — only orders 1001 (Jan, 1200), 1003 (Feb,
   60), 1006 (Mar, 950) qualify, so the three-point line in Step 2 is a
   direct consequence of that added clause, not a separate hidden query.
3. The State map's hover-highlight (Step 3) never adds a `WHERE` clause at
   all — it's the cheap, no-requery highlight mechanism from Module 5,
   which is why hovering California can visually tie back to the West bar
   instantly, with no re-aggregation of Sales figures underneath it.
4. Deselecting East (Step 4) removes the action's injected clause, and every
   affected sheet's query reverts to its unfiltered form — this "removing a
   WHERE clause and re-running" model is also exactly why the filtered
   totals in Step 2 can be independently verified by hand against `Orders`:
   they're not a Tableau-only artifact, they're the same arithmetic a raw
   SQL query with that WHERE clause would produce.

## Cheat sheet — project checklist

| Requirement | Verified by |
|---|---|
| 4 worksheets built with correct totals | Hand-computed totals in Step 1 |
| Dashboard assembled with title + 2×2 layout | Visual check |
| Global Region filter applied to all sheets | Filter control changes all 4 sheets |
| Filter action: Region bar → Time + Ratio charts | Click East, verify Step 4.2 |
| Highlight action: State map → Region bar (hover) | Hover California, verify Step 4.3 |

## Exercise

Extend the dashboard with a fifth worksheet: a text table listing Order ID,
Product, and the `Order Size` calculated field from Module 6, sorted by
Sales descending. Add it to the dashboard and include it as an additional
target of the Region filter action from Step 3, then verify by hand that
selecting Central shows exactly two rows (orders 1004 and 1007).
