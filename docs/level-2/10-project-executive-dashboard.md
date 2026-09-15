---
description: "Project — Multi-Source Executive Dashboard — This capstone combines every Level 2 module into one deliverable: a multi-source executive dashboard over…"
---

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

## How It Actually Works

This capstone's headline numbers depend on several mechanisms from earlier
Level 2 modules composing correctly — verifying it end-to-end means
checking each mechanism did what it should, not just that the final numbers
look plausible:

1. The **cross-database join** (Section 2.1) means Attainment is computed
   inside a *single* VizQL query against the joined result — `SELECT
   Region, SUM(Sales)/SUM(Target) FROM (Orders JOIN Targets ON Region)
   GROUP BY Region` conceptually — rather than two independently-rendered
   sheets whose numbers are only visually adjacent (Level 2 Module 7,
   Section 4). This is exactly why 2210/2000=1.105 can be trusted as a
   single computed field rather than something a viewer has to mentally
   divide across two charts.
2. **Extract aggregation** (Section 2.3) intentionally splits into *two*
   physical extracts here because the Region/Category sheets and the
   row-level Discount Tier table need different grains — an aggregated
   extract collapsing to Region/Category totals cannot serve the
   Discount Tier detail table's per-Order `IF` logic (Level 2 Module 1),
   since that row-level information no longer exists in an aggregated
   `.hyper` file. Verifying both extracts independently report `SUM(Sales)
   = 6880` (Section 2.3) confirms the aggregation step didn't silently drop
   or double-count rows during that collapse.
3. **Overall Attainment's two computation paths** (this module's Exercise)
   are a direct test of aggregation order: `SUM(Sales)/SUM(Target)` computes
   one ratio from two grand totals (6880/6500 = 1.0585), while the
   Sales-weighted average of per-region ratios (`Σ(Sales_i × Attainment_i) /
   ΣSales_i`) mathematically reduces to the exact same expression once
   expanded — `Σ(Sales_i × Sales_i/Target_i)` isn't generally equal to
   `ΣSales_i × ΣSales_i/ΣTarget_i`, so the fact that this dataset's numbers
   come out matching (~105.8% either way) is a coincidence of this specific
   data, not a general algebraic identity — worth flagging explicitly, since
   students who assume the two methods are *always* equivalent for any
   dataset will get burned on a differently-shaped one.
4. **The Show/Hide detail table and the filter action share the same
   underlying data**, so clicking "East" on the Region chart and separately
   expanding the Discount Tier table should show mutually consistent
   detail — Furniture+Office Supplies (2150+60=2210) on one sheet and
   exactly the East-region Order IDs (1001, 1003, 1006) on the other — any
   divergence between them indicates the filter action's field binding or
   the extract's grain doesn't match across the two sheets.

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
