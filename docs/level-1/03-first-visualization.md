---
description: "Building Your First Visualization — With Orders connected (Module 2), it's time to build an actual worksheet. This module walks through the drag-and-drop…"
---

# 03 · Building Your First Visualization

With `Orders` connected (Module 2), it's time to build an actual worksheet.
This module walks through the drag-and-drop mechanics — Columns/Rows
shelves, the Marks card, and the **Show Me** panel — by building a bar chart
of total Sales by Region, then verifying the numbers against the hand
calculation from Module 1's exercise.

## 1. From Data Source to Worksheet

1. Click the **Sheet 1** tab at the bottom of the screen (or **Worksheet >
   New Worksheet**) to leave the Data Source page and reach a blank canvas.
2. The **Data pane** on the left lists every field from `Orders`, split into
   **Dimensions** (Region, Category, Product, Order Date — blue) and
   **Measures** (Sales, Quantity, Profit — green), matching the roles from
   Module 1.
3. The main canvas is empty except for the **Columns** and **Rows** shelves
   along the top, and the **Marks card** on the left showing the current
   mark type (defaults to **Automatic**).

## 2. Building total Sales by Region (bar chart)

1. Drag **Region** from the Data pane onto the **Columns** shelf. Tableau
   adds a header for each distinct region (Central, East, West) along the
   top of the canvas — nothing is plotted yet since there's no measure.
2. Drag **Sales** from the Data pane onto the **Rows** shelf. Tableau
   automatically aggregates it as **SUM(Sales)** (the default aggregation
   for a measure) and draws one bar per region, since the Marks card mark
   type auto-switched to **Bar**.
3. Read the bar heights against your Module 1 hand calculation: East should
   total 2210 (1200 + 60 + 950), West should total 3750 (450 + 2200 + 1100),
   Central should total 920 (800 + 120). Hover over any bar to see the exact
   value in the **tooltip** Tableau generates automatically.
4. If the bars aren't sorted by height, click the **sort descending**
   icon that appears in the toolbar above the canvas (or on the Region axis
   header on hover) to rank regions by total Sales at a glance.

## 3. The Show Me panel

1. Click **Show Me** (top-right of the window, a small grid icon) to open a
   panel of chart-type thumbnails. Tableau greys out any chart type that
   doesn't fit the fields currently on your shelves and highlights ones that
   do — with Region (dimension) and SUM(Sales) (measure) already placed, the
   **bar chart** thumbnail is highlighted as the best match.
2. Show Me is a shortcut, not a requirement — everything it produces you
   could also build manually via Columns/Rows/Marks, which is why this
   module built the bar chart by hand first. Use Show Me once you already
   know roughly what chart you want and want Tableau to place the fields for
   you.
3. Try clicking a different highlighted thumbnail (e.g. horizontal bars) to
   see Tableau instantly rebuild the same view in a new orientation —
   confirming that the underlying data reference doesn't change, only the
   shelf placement.

## 4. The Marks card in more depth

1. The Marks card has a dropdown at the top (mark type: Automatic, Bar,
   Line, Circle, Shape, and more) and rows below it: **Color**, **Size**,
   **Label**, **Detail**, **Tooltip**.
2. Drag **Category** onto **Color** on the Marks card. Each region's single
   bar splits into a stacked bar of segments, one color per category
   (Furniture, Electronics, Office Supplies) — a fast way to add a second
   dimension without changing shelves.
3. Drag **SUM(Profit)** onto **Label**. Each stacked segment now shows its
   profit value directly on the mark — useful for a quick presentation view,
   though it can get visually noisy with more categories or regions than
   this example.
4. Remove a field from any shelf or the Marks card at any time by dragging
   it off the shelf (or right-click → **Remove**) — nothing here is
   destructive to the underlying data, only to this worksheet's view.

## 5. Aggregation matters

1. Right-click **SUM(Sales)** on the Rows shelf → note the **Measure**
   submenu lists other aggregations: Average, Median, Count, Minimum,
   Maximum, and more.
2. Switch it to **Average**. The East bar now shows 736.67 (2210 / 3 orders)
   instead of 2210 — the same field, a completely different story, which is
   why confirming the aggregation shown (SUM vs AVG) is the first thing to
   check when a number on a real dashboard looks surprising.
3. Switch back to **Sum** before moving on, since Module 4 assumes SUM(Sales)
   as the baseline.

## How It Actually Works

Right-clicking SUM(Sales) → **Measure** → **Average** doesn't relabel the
existing bar — it rewrites the aggregate function inside VizQL's generated
query from `SUM(Sales)` to `AVG(Sales)` and re-runs it, which is why the
whole chart, including its axis scale, redraws rather than just its label:

1. For the East bar: `SUM(Sales)` over the three East rows (1200, 60, 950)
   is 2210 — the query pattern is `SELECT Region, SUM(Sales) FROM Orders
   WHERE Region='East' GROUP BY Region`. Switching to `AVG(Sales)` changes
   the generated aggregate to `SELECT Region, AVG(Sales) FROM Orders WHERE
   Region='East' GROUP BY Region`, which evaluates to 2210 / 3 = 736.67 — a
   different SQL function over the *same* underlying rows, not a
   post-processing division applied to the displayed bar.
2. This distinguishes a **regular aggregation switch** (this module) from a
   **table calculation** (Module 8): AVG here is computed by the query
   engine itself, from the raw rows, in one pass — it isn't dependent on
   what else is in the view. A table calc, by contrast, runs *after* the
   aggregate query returns, operating only on the numbers already visible —
   which is why Module 8 needs "Compute Using" and this module doesn't.
3. Dragging **Category** onto Color (Exercise) adds `Category` to the
   `GROUP BY` clause without moving it to a shelf that produces new
   header rows — Color is VizQL's way of saying "subdivide each existing
   mark by this dimension, encoded visually, not as a new axis position."
   The underlying query becomes `GROUP BY Region, Category`, producing one
   mark per (Region, Category) pair, each colored distinctly, stacked or
   placed within its Region's bar depending on the mark type.

## Cheat sheet

| Action | How |
|---|---|
| Add a dimension to slice the view | Drag to Columns or Rows |
| Add/aggregate a measure | Drag to Columns or Rows (defaults to SUM) |
| Change chart type quickly | Show Me panel (top right) |
| Change chart type manually | Marks card dropdown (Automatic/Bar/Line/…) |
| Split bars by a second dimension | Drag dimension onto Color |
| Show values on marks | Drag measure onto Label |
| Change a measure's aggregation | Right-click field on shelf → Measure |
| See exact value | Hover for tooltip |

## Exercise

Build the Region × Sales bar chart as described, then add **Category** to
Color and **SUM(Quantity)** to Label instead of Profit. Identify, from the
resulting labels, which single (region, category) combination sold the
highest quantity of items — then verify it against the raw `Orders` table
from Module 1 by hand.
