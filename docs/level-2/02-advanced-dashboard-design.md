# 02 · Advanced Dashboard Design & Interactivity

Level 1 Module 5 built a single static dashboard. This module adds
**interactivity** — actions, containers, and device layouts — using the
8-row `Orders` table (Level 1 Module 1) as the worked example throughout.

## 1. Recap: the `Orders` table

| Order ID | Order Date | Region | Category | Product | Sales | Quantity | Profit |
|---|---|---|---|---|---|---|---|
| 1001 | 2024-01-05 | East | Furniture | Desk | 1200 | 2 | 180 |
| 1002 | 2024-01-12 | West | Electronics | Monitor | 450 | 3 | 90 |
| 1003 | 2024-02-03 | East | Office Supplies | Paper | 60 | 10 | 18 |
| 1004 | 2024-02-20 | Central | Furniture | Chair | 800 | 4 | 96 |
| 1005 | 2024-03-01 | West | Electronics | Laptop | 2200 | 2 | 330 |
| 1006 | 2024-03-15 | East | Furniture | Bookcase | 950 | 1 | -40 |
| 1007 | 2024-03-22 | Central | Office Supplies | Binders | 120 | 8 | 42 |
| 1008 | 2024-04-02 | West | Furniture | Desk | 1100 | 1 | 150 |

Region totals (hand-summed, used throughout): East = 1200+60+950 = **2210**,
West = 450+2200+1100 = **3750**, Central = 800+120 = **920**. These three sum
to 6880, the grand total — the number every dashboard filter below must
still reconcile to.

## 2. Floating vs. tiled containers

1. Tiled layout (default) stacks objects edge-to-edge in a grid that
   resizes together — safest for a first dashboard.
2. **Floating** containers let one object sit *on top of* another — e.g. a
   KPI card floating over the corner of a map. Drag an object while holding
   nothing special; instead, toggle **Floating** in the Layout pane, or hold
   `Shift` while dragging from the Dashboard pane's asset list.
3. Build a floating **title bar**: a text object reading `Regional Sales —
   $6,880 Total` floated at the top, height 40px, with a solid background
   color so it reads as a header rather than a worksheet.

## 3. Dashboard actions: Filter, Highlight, URL

1. **Filter action** (Dashboard → Actions → Add Action → Filter): clicking a
   Region bar in a bar chart filters a second sheet (e.g. a Category
   breakdown) to that Region. Verify: clicking "East" filters the Category
   sheet to show only Furniture (1200+950=2150) and Office Supplies (60) —
   summing to 2210, matching East's total exactly.
2. **Highlight action**: instead of filtering rows out, this dims
   non-matching marks — useful when you want the West region's Laptop
   order (2200) to visually pop without removing the other seven rows from
   view.
3. **URL action**: right-click a mark → configure a URL action that opens
   `https://example.com/orders/<Order ID>` using the `<Order ID>` field
   parameter — a common pattern for linking a dashboard mark back to a
   source system's record.
4. Run/Target settings: set the source sheet's action to run **On Select**
   (click) rather than **On Hover**, so accidental mouse-overs don't
   trigger filtering — a frequent build mistake for first dashboards.

## 4. Show/Hide containers for progressive disclosure

1. Add a container (e.g. a detail table) and mark it **Hide** in the
   Layout pane's "Add Show/Hide Button" option — this places a small
   toggle icon on the dashboard.
2. Use case: keep the top-level dashboard to the three Region totals
   (2210 / 3750 / 920) visible by default, and let a user click the toggle
   to reveal the full 8-row order detail table underneath, without
   consuming screen space until requested.

## 5. Device-specific layouts

1. Dashboard → **Device Layouts** → **Add Phone Layout** copies the
   Default layout as a phone-optimized starting point.
2. On the phone layout, drag underused objects (e.g. a legend) off-canvas
   or stack sheets vertically instead of side-by-side — a bar chart
   showing all three regions stacked reads better on a narrow screen than
   the 3-column desktop layout.
3. Tableau auto-detects screen width when the workbook is published and
   viewed on a phone browser or the Tableau Mobile app, switching to
   whichever device layout matches.

## 6. Verifying interactivity end-to-end

1. Build: Sheet A = bar chart of `SUM(Sales)` by Region (three bars: East
   2210, West 3750, Central 920). Sheet B = table of `Order ID`, `Product`,
   `Sales` filtered by whatever Sheet A's filter action passes.
2. Click the West bar. Sheet B should show exactly 3 rows: Monitor 450,
   Laptop 2200, Desk 1100 — summing to 3750, matching the West bar's
   height. This arithmetic check (row-level detail sums back to the
   clicked aggregate) is the standard way to confirm a filter action wired
   correctly, without needing Tableau itself running.

## How It Actually Works

Every interactivity feature in this module maps to a specific point in the
client-side rendering / query pipeline, not a uniform "dashboard magic":

1. **Filter vs. Highlight, at the query level**: a filter action always
   triggers a fresh query with an added `WHERE` clause against the target
   sheet's data source — clicking "East" re-runs the Category sheet's query
   as `SELECT Category, SUM(Sales) FROM Orders WHERE Region='East' GROUP BY
   Category`, returning exactly Furniture (2150) and Office Supplies (60),
   summing to 2210. A highlight action never touches the query layer at
   all — it operates purely on the already-rendered mark list client-side,
   toggling an opacity/color property per mark based on whether that mark's
   underlying dimension values match the selection, which is why it's
   effectively instantaneous even against a slow live source.
2. **On Select vs. On Hover** (Section 3.4) controls which browser/app
   event triggers the action's re-evaluation — On Hover binds to a
   `mousemove`-equivalent, re-firing (and re-querying, for filter actions)
   on every pixel of mouse movement across a mark, which is precisely why
   it's discouraged for filter actions: a filter action fires a real query,
   and firing one per mouse-move is needlessly expensive compared to firing
   once per deliberate click.
3. **Show/Hide containers** (Section 4) don't destroy or rebuild the hidden
   sheet's query — the underlying worksheet's query has typically already
   run (or runs lazily on first reveal, depending on the "Run all sheets"
   setting), and the toggle only changes a CSS-like visibility flag on that
   container. This is why revealing a hidden detail table is usually
   instant, not a fresh multi-second query — the cost was paid at dashboard
   load, not at toggle time.
4. **Device layouts** (Section 5) never change what's queried — the same
   worksheet's `GROUP BY` and totals (Sales 2210/3750/920) are identical
   across desktop and phone layouts. What changes is purely the rendering
   layout tree the browser/app lays those same mark results into, selected
   by matching the viewing device's reported width against the layouts
   defined in the workbook.

## Cheat sheet

| Feature | Where |
|---|---|
| Floating container | Layout pane → toggle Floating, or Shift-drag |
| Filter action | Dashboard → Actions → Add Action → Filter |
| Highlight action | Dashboard → Actions → Add Action → Highlight |
| URL action | Dashboard → Actions → Add Action → URL |
| Show/Hide button | Layout pane → Add Show/Hide Button |
| Phone layout | Dashboard → Device Layouts → Add Phone Layout |

## Exercise

Using the `Orders` table, hand-verify what a Filter action clicking
"Central" should pass through to a detail sheet: list the matching rows
(Order ID, Product, Sales) and confirm they sum to 920, Central's region
total computed in Section 1.
