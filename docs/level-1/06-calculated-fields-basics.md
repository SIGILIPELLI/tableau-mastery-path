---
description: "Calculated Fields Basics — 4. Click OK. The new field appears in the Data pane under Measures (green, since it's a numeric aggregation formula) — drag it…"
---

# 06 · Calculated Fields Basics

**Calculated fields** let you create new fields from existing ones using
formulas — a derived measure like Profit Ratio, or a derived dimension like
a size category. This module writes and hand-verifies three calculated
fields against the `Orders` table (Module 1).

## 1. Creating a calculated field

1. In the Data pane, click the dropdown arrow (top-right of the pane) or
   right-click any blank area → **Create Calculated Field...**. You can also
   right-click an existing field (e.g. Sales) → **Create > Calculated
   Field...** to start from it.
2. This opens the **calculation editor**: a Name box at top, a formula
   text area below with autocomplete for field names and functions, and an
   error indicator (bottom-left) that turns green once the formula is
   syntactically valid.
3. Name the first field `Profit Ratio` and enter:

    ```
    SUM([Profit]) / SUM([Sales])
    ```

4. Click **OK**. The new field appears in the Data pane under Measures
   (green, since it's a numeric aggregation formula) — drag it onto any
   shelf like any other measure.

## 2. Verifying Profit Ratio by hand

1. Drag **Region** to Rows and **Profit Ratio** to Text (or Rows) on a new
   sheet. Format it as a percentage: right-click the pill → **Format...** →
   set **Numbers** to **Percentage** in the Pane pane.
2. Check East by hand: SUM(Profit) for East = 180 + 18 - 40 = 158.
   SUM(Sales) for East = 1200 + 60 + 950 = 2210. Profit Ratio = 158 / 2210 ≈
   **7.15%**. Confirm the worksheet shows the same value (allowing for
   rounding in the displayed decimal places).
3. This SUM-of-SUMs pattern (aggregate first, then divide) is the correct
   way to compute a ratio in Tableau — it matches "total profit over total
   sales for the group," not an average of each row's individual ratio,
   which would overweight low-Sales orders. This distinction matters more
   once you reach Level 2's advanced calculated fields.

## 3. A row-level calculated field: Order Size

1. Create a second calculated field, `Order Size`, using **IF/ELSEIF** logic
   evaluated per row rather than per aggregate:

    ```
    IF [Sales] >= 1000 THEN "Large"
    ELSEIF [Sales] >= 300 THEN "Medium"
    ELSE "Small"
    END
    ```

2. This produces a **dimension** (Tableau colors it blue in the Data pane)
   since the result is a string, not aggregated. Drag it to Rows alongside
   Region to see order counts by size bucket per region.
3. Verify against `Orders` by hand: Order 1001 (Sales 1200) → Large. Order
   1003 (Sales 60) → Small. Order 1004 (Sales 800) → Medium. Walk all 8 rows
   this way and confirm your bucket counts match the worksheet's.

## 4. A simple arithmetic field: Profit per Unit

1. Create `Profit per Unit`:

    ```
    SUM([Profit]) / SUM([Quantity])
    ```

2. Verify for the Bookcase order alone (Product on Rows, filtered to
   Bookcase): Profit = -40, Quantity = 1, so Profit per Unit = **-40.00** —
   a loss of $40 on every unit sold, worse in per-unit terms than its raw
   Profit number alone suggests, since it sold only one unit.
3. Compare against the Desk-East order (Sales 1200, Profit 180, Quantity 2):
   Profit per Unit = 180 / 2 = **90.00** — a healthy per-unit margin despite
   a smaller total Profit than some other orders, illustrating why a
   per-unit or ratio field often tells a different story than the raw
   totals it's built from.

## 5. Editing and organizing calculated fields

1. Double-click any calculated field in the Data pane to reopen its editor
   and adjust the formula — every worksheet using it updates automatically,
   since it's one shared definition, not a copy per sheet.
2. Right-click a calculated field → **Properties** shows every worksheet in
   the workbook currently referencing it — useful before editing or deleting
   one in a workbook with many sheets, so you know what else changes.
3. Group related calculated fields into a Data pane **folder**: select
   several fields (⌘/Ctrl-click), right-click → **Group by Folder** — a
   pure organization aid with no effect on the calculations themselves.

## How It Actually Works

Whether a calculated field is computed **row-by-row before aggregation** or
**as part of the aggregate query itself** depends entirely on where the
aggregation function sits inside its formula — and that distinction is the
real mechanism behind Section 2's warning about ratios:

1. `Profit Ratio` (`SUM([Profit]) / SUM([Sales])`) is an **aggregate
   calculation**: VizQL folds it directly into the generated `SELECT`
   clause as `SUM(Profit) / SUM(Sales)`, computed once per `GROUP BY` group
   *after* every row in that group has been summed. For East: SUM(Profit) =
   180 + 18 + (-40) = 158, SUM(Sales) = 1200 + 60 + 950 = 2210, so the query
   returns 158/2210 ≈ 7.15% in one step — matching Section 2's hand check.
2. `Order Size` (the IF/ELSEIF formula with no aggregation function inside)
   is a **row-level calculation**: Tableau evaluates it once per underlying
   row, *before* any `GROUP BY` — effectively as a computed column added to
   the query's `SELECT` list ahead of grouping, e.g. conceptually `CASE WHEN
   Sales >= 1000 THEN 'Large' ... END AS OrderSize`. This is exactly why it
   comes back as a dimension (blue) rather than a measure: its result is one
   value per row, groupable, not an aggregate needing further summarization.
3. The mechanical reason a naive "average of each row's ratio" is wrong
   (Section 2.3): computing `Profit/Sales` per row *then* averaging would
   require the row-level pattern (like Order Size), producing `AVG(Profit /
   Sales)` — a completely different query shape from `SUM(Profit) /
   SUM(Sales)`, and one that weights a $60 order's ratio equally with a
   $2200 order's ratio instead of by dollar volume. Hand-check: East's
   naive average would be (180/1200 + 18/60 + -40/950)/3 ≈ (0.15 + 0.30 +
   -0.042)/3 ≈ 13.6% — nowhere near the correct 7.15%, because it lets a
   small, high-margin order dominate the average.

## Cheat sheet

| Action | How |
|---|---|
| Create a calculated field | Data pane dropdown → Create Calculated Field... |
| Aggregate ratio (correct pattern) | `SUM([A]) / SUM([B])` |
| Row-level conditional field | `IF ... THEN ... ELSEIF ... ELSE ... END` |
| Edit an existing calculation | Double-click it in the Data pane |
| See what uses a calculated field | Right-click → Properties |
| Format as percentage | Right-click pill → Format... → Numbers → Percentage |

## Exercise

Create a calculated field `Is Loss` that returns `TRUE` if a row's Profit is
negative and `FALSE` otherwise, using an IF/THEN/ELSE formula. Drag it to
Rows alongside Product and confirm by hand, from the `Orders` table, that
exactly one product (Bookcase) evaluates to `TRUE`.
