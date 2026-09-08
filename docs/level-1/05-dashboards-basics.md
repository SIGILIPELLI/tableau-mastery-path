# 05 · Building Dashboards Basics

A **dashboard** combines multiple worksheets into one interactive view, with
actions letting one chart filter or highlight another. This module builds a
two-sheet dashboard from the Region-by-Sales bar chart (Module 3) and the
Sales-over-time line chart (Module 4), then wires a filter action between
them.

## 1. Creating a dashboard

1. Click the **New Dashboard** icon at the bottom of the window (next to the
   worksheet tabs), or **Dashboard > New Dashboard**. This opens a blank
   canvas with a **Size** control (top-left of the Dashboard pane) —
   leave it at the default **Desktop Browser** size for this exercise.
2. The left **Dashboard pane** lists every worksheet in the workbook —
   from Modules 3 and 4, this includes your Region bar chart and your
   Sales-over-time line chart (rename each sheet tab descriptively first,
   e.g. double-click **Sheet 1** → "Sales by Region", so the dashboard pane
   is readable).
3. Drag "Sales by Region" from the pane onto the left half of the dashboard
   canvas. Drag "Sales Over Time" onto the right half. Tableau's **Tiled**
   layout (the default) auto-arranges them side by side with a divider you
   can drag to resize.

## 2. Titles, text, and legends

1. Drag a **Text** object (from the **Objects** section at the bottom of the
   Dashboard pane) above the two charts, double-click it, and type a
   dashboard title like "Northwind Retail — Sales Overview".
2. If a legend (e.g. for Category color, if you kept that from Module 3)
   appears cluttered or floating oddly, drag its edge to reposition it, or
   right-click it on the dashboard → **Remove from Dashboard** if it's
   redundant with a chart's own labels.
3. Toggle between **Tiled** and **Floating** per-object via right-click on
   an object's grey border → **Floating** — floating objects can overlap and
   be positioned freely (useful for a title banner or a logo), while tiled
   objects respect the grid and resize predictably as the dashboard resizes.

## 3. Filter actions — making one chart filter another

1. Go to **Dashboard > Actions... > Add Action > Filter**.
2. Set **Source Sheets** to "Sales by Region" and **Target Sheets** to
   "Sales Over Time". Set **Run action on** to **Select** (the default),
   and leave **Target Filters** as **All Fields** (Tableau will filter the
   target on the dimensions shared between the two sheets — here, Region).
3. Click **OK**, then test it: on the dashboard, click the **East** bar in
   "Sales by Region". The line chart should update to show only East's
   monthly Sales trend — the pattern behind almost every "click a bar,
   watch other charts update" dashboard you've ever used.
4. Click the East bar again (or an empty area of the chart) to **deselect**
   and return the line chart to showing all regions — filter actions based
   on **Select** toggle off when you click the same mark again.

## 4. Highlight actions — a softer alternative

1. Add a second action: **Dashboard > Actions... > Add Action > Highlight**.
   Same Source/Target sheet setup as the filter action.
2. Unlike a filter action, a highlight action doesn't remove data from the
   target view — it dims everything *except* the matching marks, keeping
   context (you can still see all regions' magnitude in the target) while
   drawing attention to the selected one.
3. Choose Filter vs. Highlight based on intent: use **Filter** when the
   target view would be genuinely more useful showing only the selection
   (e.g. drilling from a Region bar into that region's own trend), and
   **Highlight** when preserving the full picture while calling out the
   selection matters more (e.g. showing where one region ranks among all
   of them over time).

## 5. Dashboard-level filters (Show as Filter)

1. Right-click the **Region** field's legend or a sheet's Region filter
   card and choose **Apply to Worksheets > All Using This Data Source** to
   make one filter control every sheet in the workbook that shares the
   `Orders` connection, rather than wiring per-sheet actions for a case this
   simple.
2. This is the fastest way to add a single global filter dropdown; actions
   (Section 3–4) are for the richer "clicking a mark drives another chart"
   interactivity that a simple filter control can't express.

## How It Actually Works

A dashboard doesn't merge its sheets into one query — each tile keeps
issuing its own independent VizQL query, and an **action** works by
capturing a click/hover event and injecting an implicit filter clause into
the *other* sheets' next query run:

1. A filter action from "Sales by Region" to "Sales Over Time" means: when
   you click the East bar, Tableau appends `WHERE Region = 'East'` to the
   query that "Sales Over Time" would otherwise run, then re-executes it.
   That's why the line chart's re-drawn totals must equal a hand-filter of
   `Orders` to East rows only (1001, 1003, 1006) — it's the same underlying
   `GROUP BY Month` query, just with an added `WHERE` clause supplied by the
   action, not a different chart.
2. **Highlight actions** are cheaper than filter actions mechanically: they
   don't add a `WHERE` clause or re-run the query at all — they only change
   which already-rendered marks get dimmed vs. full-opacity client-side,
   which is why hovering to highlight is instantaneous even against a live,
   slow data source, while a filter action re-triggers a real query round
   trip.
3. **"Apply to Worksheets > All Using This Data Source"** (Section 5) is
   different from both: it doesn't operate as a click-triggered action at
   all, but as a shared filter clause baked into every sheet's query from
   the same connection, evaluated before the sheet even renders — which is
   why it needs no "clicking a mark" step and instead behaves like a
   dashboard-wide global `WHERE` a viewer's dropdown directly controls.

## Cheat sheet

| Task | Where |
|---|---|
| Create a dashboard | Bottom tab bar → New Dashboard icon |
| Add a sheet to the dashboard | Drag from Dashboard pane onto canvas |
| Add a title/text | Objects section → Text, drag onto canvas |
| Tiled vs Floating object | Right-click object border |
| Configure filter/highlight actions | Dashboard menu → Actions... |
| Global filter across all sheets | Right-click filter card → Apply to Worksheets |

## Exercise

Build the two-sheet dashboard described above with a working filter action
from "Sales by Region" to "Sales Over Time". Then add a third worksheet
(a simple table: Category on Rows, SUM(Profit) on Text) to the dashboard,
and extend the same filter action so clicking a region bar also filters the
Category/Profit table.
