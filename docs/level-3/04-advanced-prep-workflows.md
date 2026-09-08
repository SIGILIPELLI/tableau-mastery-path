# 03 · Advanced Tableau Prep Workflows

Level 2 introduced Tableau Prep for basic cleaning. This module builds a
multi-step Prep flow against an expanded version of `Orders` that includes
messy real-world artifacts, and hand-verifies every output.

## 1. The messy source: `Orders_Raw`

`Orders_Raw` is the same 8 orders as the Level 1 `Orders` table, but as it
would actually arrive from an export — extra whitespace, inconsistent
casing, and a `Shipping Cost` column that's a text field with a currency
symbol:

| Order ID | Region | Category | Sales | Shipping Cost |
|---|---|---|---|---|
| 1001 | " East" | furniture | 1200 | "$24.00" |
| 1002 | "West " | Electronics | 450 | "$9.00" |
| 1003 | "East" | OFFICE SUPPLIES | 60 | "$3.00" |
| 1004 | "Central" | Furniture | 800 | "$16.00" |
| 1005 | "West" | electronics | 2200 | "$44.00" |
| 1006 | "East" | Furniture | 950 | "$19.00" |
| 1007 | "Central" | office supplies | 120 | "$2.40" |
| 1008 | " West" | furniture | 1100 | "$22.00" |

## 2. Flow step 1: Clean

1. **Trim whitespace**: apply Prep's auto-suggested "Trim" transform to
   `Region` — this strips the leading/trailing spaces from " East", "West
   ", and " West", leaving `East`, `West`, `West`.
2. **Standardize case**: apply "Change Case → Proper Case" to `Category` —
   `furniture` → `Furniture`, `OFFICE SUPPLIES` → `Office Supplies`,
   `electronics` → `Electronics`. After this step, grouping by `Category`
   collapses to exactly 3 distinct values instead of 6 (furniture,
   Furniture, OFFICE SUPPLIES, Office Supplies, electronics, Electronics
   all present pre-clean).
3. **Parse currency text to number**: `Shipping Cost` arrives as a string
   with a `$`. Use a calculated field `Shipping Cost (Num)` =
   `FLOAT(REPLACE([Shipping Cost], "$", ""))`. Verify by hand: `"$24.00"` →
   remove `$` → `"24.00"` → FLOAT → `24.0`. Do this for all 8 rows; the sum
   should be 24+9+3+16+44+19+2.4+22 = **139.4**.

## 3. Flow step 2: Aggregate

1. Add an **Aggregate** step grouping by cleaned `Region`, summing `Sales`
   and `Shipping Cost (Num)`.
2. Hand-compute the expected output:
   - East: Sales 1200+60+950=2210, Shipping 24+3+19=46
   - West: Sales 450+2200+1100=3750, Shipping 9+44+22=75
   - Central: Sales 800+120=920, Shipping 16+2.4=18.4
3. Check the totals: Sales 2210+3750+920 = 6880 (matches the Level 1
   grand total — cleaning didn't change any Sales values, only text
   formatting). Shipping: 46+75+18.4 = 139.4 (matches Section 2.3).

## 4. Flow step 3: Pivot for a wide output

1. Suppose a downstream report wants one row per Region with a column per
   Category's Sales, rather than one row per Region+Category. Use Prep's
   **Pivot (rows to columns)** step, pivoting `Category` into column
   headers with `Sales` as the value.
2. Expected wide table:

    | Region | Furniture | Electronics | Office Supplies |
    |---|---|---|---|
    | East | 2150 | 0 (blank) | 60 |
    | West | 1100 | 2650 | (blank) |
    | Central | 800 | (blank) | 120 |

3. Verify each row sums back to the Region total from Section 3.2: East
   2150+60 = 2210 ✓. West 1100+2650 = 3750 ✓. Central 800+120 = 920 ✓.

## 5. Output step and incremental refresh

1. The **Output** step writes the flow's result to a `.hyper` extract or
   back to a database table. For a flow that runs nightly against a
   growing `Orders_Raw` table, enable **incremental refresh** keyed on
   `Order ID`, so Prep only processes new/changed rows on each run rather
   than reprocessing all 8 (or, in production, all N) rows every time.
2. Incremental refresh requires the key column to be reliably
   increasing/new-row-only (an Order ID that's never reused or edited
   after creation) — editing a historical row's Sales value would not be
   picked up by an incremental refresh keyed this way, which is the
   trade-off for the speed gain.

## How It Actually Works

Building on Level 2 Module 6's node-pipeline model, this flow's Pivot and
Incremental Refresh steps expose two mechanisms worth tracing precisely:

1. **Pivot (rows to columns) as a physical regrouping**: the step doesn't
   compute anything new arithmetically — it takes the already-aggregated
   Region×Category×Sales rows from Section 3 and re-projects the Category
   dimension's distinct values into column headers, placing each existing
   Sales value into the cell matching its (Region, Category) pair, with a
   blank/0 wherever no matching row existed (East has no Electronics
   orders, so that cell is blank, not a computed zero from missing data).
   This is why every row's non-blank cells sum back exactly to that row's
   pre-pivot Region total (2150+60=2210 for East) — the pivot step
   redistributes existing summed values into a wider row shape, it never
   re-aggregates or drops values in the process.
2. **Incremental refresh keyed on Order ID**: mechanically, this changes
   the Input node's extraction query from "read all rows" to "read only
   rows whose key exceeds the highest key value processed on the last run"
   — a `WHERE OrderID > <last_max_id>`-style predicate, conceptually
   identical to an extract filter (Level 2 Module 8) but computed
   automatically per run rather than hand-specified. This is exactly why
   an edited *historical* row (Order ID already below the last-processed
   watermark) is invisible to this refresh mode: the generated predicate
   structurally excludes any row whose key isn't newly above the
   watermark, regardless of whether its other columns changed.
3. **Why Clean steps must run before Aggregate, and Aggregate before
   Pivot, in this flow specifically**: each node consumes the *exact* row
   set the previous node emits (Level 2 Module 6's chained-CTE model) — an
   Aggregate step run before the Clean/case-fix step would group `Furniture`
   and `furniture` as two separate keys, producing a 6-row (not 3-row)
   aggregate that a downstream Pivot step would then also incorrectly
   spread across 6 columns instead of 3, propagating the uncleaned
   granularity all the way to the final wide-table output.

## Cheat sheet

| Step | Purpose | Verified output |
|---|---|---|
| Clean (Trim, Change Case) | Fix formatting inconsistency | 3 distinct Regions/Categories |
| Calculated field (parse currency) | Text → number | Shipping total 139.4 |
| Aggregate | Group by cleaned Region | East 2210, West 3750, Central 920 |
| Pivot | Rows → columns | Wide table, rows sum to Region totals |
| Output + incremental refresh | Efficient nightly load | Keyed on non-reused Order ID |

## Exercise

`Orders_Raw` gets a 9th row: Order ID 1009, Region `" east"` (lowercase,
leading space), Category `"FURNITURE"`, Sales 300, Shipping Cost `"$6.00"`.
After the same Clean step (trim + proper case), which existing Region and
Category does this row join, and what are the new East and Furniture
totals? (Region "East" — trim removes the space, proper case would need a
capitalization rule too, but Prep's "Change Case" on Region isn't applied
in this flow, so confirm whether a separate proper-case step on Region is
needed. New East Sales = 2210+300 = 2510. New Furniture wide-column total
= 2150+300 = 2450.)
