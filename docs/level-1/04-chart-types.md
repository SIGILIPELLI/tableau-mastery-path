---
description: "Chart Types & When to Use Them — Tableau can build dozens of chart types, but most real analysis needs only a handful used well. This module covers bar…"
---

# 04 · Chart Types & When to Use Them

Tableau can build dozens of chart types, but most real analysis needs only a
handful used well. This module covers bar, line, scatter, and pie charts —
building each against `Orders` (Module 1), and critiquing when each is (and
isn't) the right choice.

## 1. Bar charts — comparing categories

1. From Module 3, you already have Region on Columns and SUM(Sales) on Rows.
   Bar charts are the right default for **comparing a measure across
   discrete categories** — regions, products, categories — because humans
   judge bar *length* very accurately.
2. Add **Category** to Rows (after Region) to get a bar per (Region,
   Category) pair, rather than stacking as color did in Module 3 — this
   "small multiples" layout lets you compare exact lengths directly, which
   stacked segments make hard (comparing a middle segment's length across
   bars requires subtracting, since stacked segments don't share a common
   baseline).
3. **When not to use a bar chart**: more than ~15–20 categories, since bars
   become too thin to label or compare meaningfully — a filtered top-N view
   or a table is often clearer at that scale (Level 2 covers filtering
   patterns like this).

## 2. Line charts — trends over time

1. Remove Category/Region from the shelves (drag off) and start fresh: drag
   **Order Date** onto Columns. Tableau adds it as a discrete date hierarchy
   by default (Year > Quarter > Month > Day, expandable via the `+` on the
   pill) — right-click the **Order Date** pill on Columns and choose
   **Month** under the **Continuous** section (not Discrete) to get a single
   continuous timeline rather than one header per hierarchy level.
2. Drag **Sales** onto Rows. The Marks card mark type auto-switches to
   **Line**, plotting SUM(Sales) per month across the `Orders` data
   (January through April 2024).
3. Line charts are the right choice for **showing a trend or change over a
   continuous field** — almost always time. A bar chart with dates on
   Columns forces one bar per date, which visually competes with the more
   natural "trajectory" a line conveys.
4. **When not to use a line chart**: connecting points that aren't naturally
   ordered/continuous (e.g. a line across unordered categories like Product
   names) implies a trend or interpolation that doesn't exist — that's
   exactly the bar chart's job instead.

## 3. Scatter plots — relationships between two measures

1. Start a new sheet. Drag **Sales** onto Columns and **Profit** onto Rows —
   with two measures and no dimension yet, Tableau plots a single mark (the
   grand total point). Drag **Product** onto **Detail** on the Marks card to
   break that single point into one mark per product (8 points, matching the
   8 rows of `Orders`, since each product here appears once).
2. Scatter plots answer **"how do two measures relate, per item?"** —
   here, whether higher Sales tends to come with higher Profit. Looking at
   the worked data: the Bookcase order (Sales 950, Profit **-40**) breaks
   that pattern — a loss on a mid-sized sale, visually an outlier below the
   rest of the cluster.
3. Drag **Category** onto **Color** to see whether the outlier belongs to a
   pattern (e.g. "Furniture tends to run thinner margins") — with only 8
   points in this course dataset, that's a hypothesis to state and check
   against more data, not a firm conclusion, which is itself a good
   scatter-plot habit to build.
4. **When not to use a scatter plot**: more than one dimension and one
   measure without a second measure to plot against — a scatter needs two
   numeric axes to make sense; a single measure against a dimension is a bar
   chart's job.

## 4. Pie charts — proportion of a whole (with a warning)

1. Show Me's pie chart thumbnail requires one dimension and one measure —
   drag **Category** onto Color and **SUM(Sales)** onto Angle (or use Show
   Me directly) to get three wedges: Furniture, Electronics, Office
   Supplies.
2. Pie charts answer **"what share of the total does each category have?"**
   — and only really work well for a **small number of categories** (2–5)
   with clearly different sizes. Here: Electronics ≈ 2650 (450+2200), 
   Furniture ≈ 3050 (1200+800+950+1100), Office Supplies ≈ 180 (60+120) —
   with only 3 wedges and one much smaller than the others, the pie
   communicates the story reasonably.
3. **The general critique**: humans compare *angles* far less accurately
   than bar *lengths*, so a pie with more than ~5 similarly-sized wedges
   becomes nearly unreadable — that same Category-by-Sales comparison as a
   bar chart (Section 1) would scale to many more categories without losing
   clarity. Default to a bar chart unless "part of a whole, few categories"
   is specifically the story you're telling.

## 5. Choosing quickly — a decision guide

- Comparing a measure across categories → **bar chart**.
- A measure changing over time → **line chart**.
- Relationship between two measures, per item → **scatter plot**.
- Share of a whole, 2–5 categories → **pie chart** (bar chart otherwise).
- Geographic patterns → **map** (Module 9).

## How It Actually Works

A "chart type" in Tableau is not a separate rendering mode with its own
query logic — every chart type in this module issues the *same style* of
`GROUP BY` aggregate query; what changes is purely how VizQL maps the
returned rows onto marks:

1. **Bar vs. line**: both a Category-bar chart and an Order-Date line chart
   generate a `SELECT <dimension>, SUM(Sales) FROM Orders GROUP BY
   <dimension>` query — the difference is that a continuous date field
   (Month, treated as continuous per Module 3's technique) is placed on an
   interpolated numeric axis and connected point-to-point, while a discrete
   dimension (Category) produces separate header positions with independent
   bars. Swap Order Date from continuous to discrete and the "line" chart
   would fall back to disconnected marks — same query, same result set,
   different mark geometry.
2. **Scatter plot**: putting a measure on both Columns and Rows (e.g.
   SUM(Sales) and SUM(Profit) per order) with a dimension on Detail
   generates one row per Order ID rather than one per Region, because
   Detail — unlike Color/Label — still forces its field into the `GROUP BY`
   even though nothing renders it visually; it exists purely to keep marks
   from being aggregated together.
3. **Pie chart**: internally still a `GROUP BY Category` aggregate, but the
   Angle shelf tells VizQL to render each group's proportion of the sum as a
   wedge angle rather than a bar length — verify by hand: Office Supplies'
   share of total Sales is (60+120) / 6880 ≈ 2.6%, a wedge so thin it's
   nearly invisible next to Electronics' (450+2200)/6880 ≈ 38.5% — the exact
   readability failure mode Section 4 describes, and the reason a pie's
   accuracy problem is perceptual (angle vs. length judgment), not a
   difference in the underlying arithmetic.

## Cheat sheet

| Question being answered | Chart type | Key shelf setup |
|---|---|---|
| Compare a measure across categories | Bar | Dimension on Columns, Measure on Rows |
| Trend over time | Line | Continuous date on Columns, Measure on Rows |
| Relationship between two measures | Scatter | Measure on Columns, Measure on Rows, dimension on Detail |
| Share of a whole (few categories) | Pie | Dimension on Color, Measure on Angle |

## Exercise

Using `Orders` (Module 1), decide which chart type you'd use to answer each
of these three questions, and justify why in one sentence each: (1) "Did
total Sales grow month over month?" (2) "Which single order had an unusually
bad Profit relative to its Sales?" (3) "What fraction of total company Sales
came from Office Supplies?"
