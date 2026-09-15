---
description: "Formatting & Storytelling with Data — A technically correct chart can still fail to communicate. This module covers formatting discipline and Tableau's…"
---

# 05 · Formatting & Storytelling with Data

A technically correct chart can still fail to communicate. This module
covers formatting discipline and Tableau's **Story Points** feature, using
the `Orders` table's Region totals (East 2210, West 3750, Central 920) as
the running example.

## 1. Number formatting that matches the audience

1. Right-click a measure pill → **Format** → set **Numbers** to Currency
   (Custom) with 0 decimal places for an executive audience: `$2,210` reads
   faster than `$2,210.00` or the raw `2210`.
2. For percentages (e.g. `Pct of Region Sales` from Level 2 Module 1),
   format as Percentage with 1 decimal: East's Bookcase share (950/2210 =
   0.4299...) should display as `43.0%`, not `0.43` or `42.99%` — 1 decimal
   is the standard precision for a share-of-total metric at this scale.
3. Use **Unit abbreviation** (thousands `K` / millions `M`) only when
   numbers are large enough to need it; forcing `$2.2K` for a $2,210 value
   on a small dashboard often reads as *less* precise, not more — a design
   judgment worth stating explicitly to students who over-apply this.

## 2. Color: sequential vs. diverging vs. categorical

1. **Categorical** palette (distinct hues) for Region: East, West, Central
   get three visually distinct, non-ordered colors — correct because
   Region has no inherent order.
2. **Sequential** palette (single hue, light→dark) for `SUM(Sales)`
   itself: darker = higher sales. Central (920) would be the lightest
   shade, West (3750) the darkest, correctly encoding magnitude.
3. **Diverging** palette (two hues meeting at a midpoint) for `Profit`,
   centered at 0: the Bookcase order's Profit (-40) should render in the
   "loss" hue while every other order (all positive Profit) renders in the
   "gain" hue — diverging color is the only palette type that visually
   flags the one negative row in this dataset.

## 3. Titles, subtitles, and annotations that carry the point

1. A worksheet title should state the *finding*, not just the field name:
   prefer `"West leads at $3,750 — 55% of total Sales"` over a generic
   `"Sales by Region"`. Verify the 55% claim: 3750/6880 = 0.545 ≈ 55%.
2. Add a mark annotation (right-click the Bookcase mark → Annotate →
   Mark) reading `"Only loss: -$40 profit"` — annotations should call out
   exceptions the audience would otherwise have to hunt for.
3. Avoid over-annotating: with only 8 rows, annotate at most the 1–2 marks
   that matter (the single loss, the single largest order) rather than
   labeling every mark, which recreates a dense table and defeats the
   purpose of a chart.

## 4. Story Points

1. **Story** is a Tableau sheet type (right-click the sheet tabs → New
   Story) that sequences multiple existing worksheets/dashboards into a
   guided narrative, each step called a **story point**.
2. Build a 3-point story on the `Orders` data: Point 1 = "Overview" (bar
   chart of the three region totals: 2210 / 3750 / 920). Point 2 = "What's
   driving West" (West's 3 orders: Monitor 450, Laptop 2200, Desk 1100 —
   the Laptop alone is 2200/3750 = 58.7% of West's total). Point 3 = "The
   one exception" (Bookcase, Profit -40, the only loss in the dataset).
3. Each story point can carry its own **caption** (below the point
   navigator) — use short, declarative captions ("West's $3,750 is driven
   by one $2,200 laptop order") rather than restating the chart title.

## 5. Layout discipline: the "5-second test"

1. A dashboard should communicate its headline number within 5 seconds of
   viewing — for this dataset, that's the $6,880 grand total and the
   $3,750 West-leads-Region finding, both of which should be the largest,
   top-left elements, not buried below a legend or filter panel.
2. Group secondary detail (the full 8-row order table) below or behind a
   Show/Hide toggle (Level 2 Module 2) rather than competing for the same
   visual weight as the headline chart.

## How It Actually Works

Formatting and color are almost entirely a **rendering-layer** concern
applied after VizQL's query has already returned its numbers — but two
mechanisms in this module do interact with the query/calc engine directly:

1. **Number formatting never changes underlying values** — it's a display
   transform applied to the same returned aggregate. East's Sales stays the
   exact integer 2210 in the query result regardless of whether it's
   rendered as `2210`, `$2,210`, or `$2.2K`; verify this distinction
   matters by noting a *calculated field* referencing `[Sales]` in a
   formula always sees the raw 2210, never a rounded or abbreviated
   version, even if every visible label on the sheet is abbreviated.
2. **Diverging color's midpoint is a genuine value comparison**, not purely
   visual: Tableau's diverging palette assigns each mark's color by
   comparing its numeric value against a computed or user-set center
   (default: the field's midpoint, or 0 if the field can meaningfully cross
   zero) — the Bookcase order's Profit (-40) is colored differently only
   because -40 < 0 is evaluated per-mark before rendering; the color
   assignment is itself a small per-mark classification computed from the
   same aggregate the axis already shows, not a separate stored field.
3. **Story Points don't create new queries** — each story point is a
   captured *reference* to an existing sheet/dashboard's current filter and
   parameter state (a saved "bookmark" of shelf/filter configuration), so
   navigating between points re-applies whichever filter/parameter state
   was captured, potentially re-triggering that sheet's query if the state
   differs from what's currently cached, but building no new calculation
   logic of its own — this is why a story point can "break" if the
   underlying sheet's fields are later renamed or removed: the story only
   holds a reference, not an independent copy.

## Cheat sheet

| Element | Rule of thumb |
|---|---|
| Currency formatting | 0 decimals unless sub-dollar precision matters |
| Percentage formatting | 1 decimal place is standard |
| Categorical color | Unordered dimensions (Region, Category) |
| Sequential color | Ordered magnitude, one direction (Sales) |
| Diverging color | Values that cross a meaningful zero (Profit) |
| Title | State the finding, not the field name |
| Story point | One narrative beat per point; short caption |

## 🔀 Related lessons on other tracks

- [Data Science — 06 · Data Storytelling for Executives](https://sigilipelli.github.io/data-science-mastery-path/level-3/06-data-storytelling-executives/)

## Exercise

Write three story-point titles (not generic chart titles) for a story built
from the `Orders` data that would each pass the "states a finding, not a
field name" test — one about the grand total, one about the West region,
and one about the Bookcase loss — and hand-verify each numeric claim you use.
