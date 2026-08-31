# 10 · Project — Multi-Source Executive Dashboard

This capstone combines every Level 2 module into one deliverable: a
multi-source executive dashboard over `Orders` and `Targets`, published and
interactive.

## 1. Inputs

`Orders` (8 rows, Level 1 Module 1) and `Targets` (Level 2 Module 7):

| Region | Sales | Target | Attainment |
|---|---|---|---|
| East | 2210 | 2000 | 110.5% |
| West | 3750 | 3500 | 107.1% |
| Central | 920 | 1000 | 92.0% |

Attainment computed as Sales/Target: 2210/2000=1.105, 3750/3500=1.0714,
920/1000=0.92 — confirmed against Level 2 Module 7's Section 3.

## 2. Data layer

1. Combine `Orders` and `Targets` via a **cross-database join** (Module 7)
   on `Region`, rather than a blend, so a single calculated field can
   compute Attainment without cross-source limitations.
2. Add `Pct of Region Sales` and `Discount Tier` from Level 2 Module 1 for
   drill-down detail: Furniture's East share example (Bookcase 950/2210 =
   43.0%) and the Tier A/B/C/Review classification per order.
3. Extract the combined source (Module 8) with aggregation enabled for the
   Region/Category-level sheets, and a separate non-aggregated extract (or
   the same extract without the aggregation option) for the row-level
   detail sheet — verify both report `SUM(Sales) = 6880` before proceeding.

## 3. Sheets to build

1. **KPI header**: three floating text/number objects — Total Sales
   ($6,880), Total Profit ($866: 180+90+18+96+330-40+42+150), Overall
   Attainment (6880/6500 = 105.8%, using Target total 2000+3500+1000=6500).
2. **Region bar chart**: `SUM(Sales)` by Region with a reference line at
   each Region's Target (dual-axis or a Gantt-bar target overlay) — West's
   bar (3750) should visibly clear its 3500 reference line; Central's bar
   (920) should fall short of its 1000 line.
3. **Category breakdown**: Furniture 4050, Electronics 2650, Office
   Supplies 180 — built as a second sheet, linked to the Region chart via
   a **filter action** (Level 2 Module 2).
4. **Discount Tier detail table**: row-level `Order ID`, `Product`,
   `Discount Tier` from Level 2 Module 1's classification (Tier A: 1001,
   1005; Tier B: 1008; Review: 1006; Tier C: 1002, 1003, 1004, 1007) — kept
   inside a Show/Hide container so it doesn't compete with the KPI header.

## 4. Interactivity checklist

1. Clicking a Region bar filters the Category sheet — verify clicking
   "East" shows Furniture (1200+950=2150) and Office Supplies (60),
   summing to 2210.
2. A Show/Hide toggle reveals the Discount Tier detail table.
3. A phone device layout stacks the KPI header above a single combined
   chart rather than the 3-column desktop arrangement.

## 5. Formatting and storytelling pass (Module 5)

1. Title reads the finding: `"West leads at $3,750 (107% of target) —
   Central trails at 92%"`.
2. Diverging color on the Attainment field, centered at 100%: West and
   East render in the "above target" hue, Central in the "below target"
   hue — correctly flagging Central as the one region under its goal.

## 6. Publish and verify

1. Publish to Tableau Server/Cloud (Module 9) as a separate data source
   plus a workbook referencing it, with "Download Full Data" restricted
   for viewer-level permissions.
2. Attach a nightly extract refresh schedule.
3. Post-publish check: open the published URL and re-confirm all headline
   numbers — Total Sales $6,880, Total Profit $866, Overall Attainment
   105.8%, and each Region's bar/target relationship — exactly match the
   hand-computed values in Sections 1–3. Any mismatch means the publish
   used a stale or mis-joined data source and should be re-published
   after fixing the join.

## Cheat sheet — capstone checklist

| Layer | Delivered |
|---|---|
| Data | Orders + Targets cross-database join, verified totals |
| KPIs | Total Sales, Total Profit, Overall Attainment |
| Charts | Region-vs-Target bars, Category breakdown, Tier detail |
| Interactivity | Filter action, Show/Hide container, phone layout |
| Formatting | Finding-based title, diverging Attainment color |
| Publish | Server publish, restricted download, refresh schedule |

## Exercise

Compute Overall Attainment a second, independent way — as the
Sales-weighted average of the three regional attainments, i.e.
`(2210×1.105 + 3750×1.0714 + 920×0.92) / 6880` — and confirm it still comes
out to approximately 105.8%, matching the Total Sales ÷ Total Target method
used in Section 3.
