---
description: "Advanced Performance Tuning — Slow dashboards usually trace to one of a handful of causes. This module diagnoses each against the Orders workbook…"
---

# 05 · Advanced Performance Tuning

Slow dashboards usually trace to one of a handful of causes. This module
diagnoses each against the `Orders` workbook, quantifying "how much work"
each fix removes using row/calculation counts you can verify by hand.

## 1. Reading the Performance Recording

1. Tableau Desktop's **Help → Settings and Performance → Start Performing
   Recording** captures a timeline of every query, calculation, layout, and
   rendering event while you interact with a workbook, then opens a
   dashboard of results when you stop recording.
2. The recording's events are sorted by duration — the top bar is almost
   always either "Executing Query" (time spent waiting on the data source)
   or "Computing Layout" (time spent rendering marks) for a workbook this
   small; for larger production workbooks, "Executing Query" typically
   dominates for live connections, "Computing Layout" for dashboards with
   many zones/filters.

## 2. Extract vs. live: quantifying the difference

1. `Orders` has 8 rows — trivially fast either way — but the pattern for
   reasoning about it scales: a live connection re-runs a query against
   the source (e.g. `SELECT Region, SUM(Sales) FROM Orders GROUP BY
   Region`) on every filter change, while an extract answers from a local
   `.hyper` copy.
2. If `Orders` had 8 million rows instead of 8, the query cost scales
   with the source engine's ability to aggregate that volume; an extract
   pre-indexes and compresses data specifically for the aggregations
   Tableau needs, so the same GROUP BY typically returns faster from a
   `.hyper` extract than from a live query hitting an un-indexed table —
   the actual multiplier depends on the source, which is why Tableau's own
   guidance is "test both," not a fixed rule.

## 3. Calculated field cost: row-level vs. aggregate vs. table calc

1. **Row-level calculations** (e.g. `[Sales] * 1.1`) execute once per row
   — 8 evaluations for `Orders`.
2. **Aggregate calculations** (e.g. `SUM([Sales])`) execute once per
   rendered mark/group — 3 evaluations if grouped by Region (East, West,
   Central).
3. **Table calculations** (e.g. a running total) execute once per mark but
   also require the *entire* partition materialized in memory to compute
   relative to other marks — for `Orders` grouped by Region, that's still
   3, but the cost of a table calc grows with partition size in a way flat
   aggregates don't, because each cell's answer depends on its neighbors.
4. Rule of thumb ranked by relative cost per row of underlying data: LOD
   FIXED calcs (computed once per unique combination, cached) ≤ aggregate
   ≤ row-level ≤ table calc ≤ LOD with non-aggregate dependency chains.

## 4. Filters: dimension filter vs. context filter vs. data source filter

1. A **data source filter** (Level 1) is applied earliest, at the query
   itself — e.g. filtering `Orders` to `Category != "Office Supplies"`
   means only 6 rows (excluding 1003, 1007) are ever pulled into the
   workbook, reducing every downstream calculation's input size from 8 to
   6 rows.
2. A **context filter** (Level 2 Module 8) is applied next, materializing
   an intermediate result other filters/calcs then run against — useful
   when a FIXED LOD needs to respect a filter (Level 3 Module 1, Section 4)
   but comes at a small recomputation cost whenever the context filter's
   own value changes.
3. A plain **dimension/measure filter** runs last, against whatever the
   context (or full data source) already produced — cheapest to change
   interactively, since it doesn't force recomputation of LODs or context.

## 5. Worked diagnosis: a slow dashboard with 4 worksheets

1. Symptom: a dashboard combining 4 worksheets (Region bar chart, Category
   pie, a table with 3 table calcs, and a map) takes 8 seconds to render
   after a filter change on a live connection.
2. Diagnostic order: (a) check Performance Recording for the dominant
   event type; (b) if "Executing Query" dominates, test converting the
   live connection to an extract, and re-measure; (c) if "Computing
   Layout" or calc time dominates instead, count how many table calcs
   recompute per filter change (3 in the table worksheet) versus how many
   are LOD FIXED calcs (which don't need to fully recompute if the filter
   isn't in their FIXED dimension list, per Module 1 Section 4) — a table
   calc-heavy worksheet recomputing across a large partition is a common
   hidden cost that an extract alone won't fix.
3. Verify the fix numerically where possible: after switching to an
   extract, the underlying `Orders` totals must stay identical (Region
   East 2210, West 3750, Central 920, grand total 6880) — performance
   tuning should never change a result, only the time to produce it; if a
   total changes after a tuning change, that's a correctness regression,
   not a performance win.

## How It Actually Works

1. The Performance Recording workbook is itself a Tableau data source: each
   event (Executing Query, Computing Layout, Compiling Query, Blending
   Data, ...) is a row with a start time and duration, drawn as a Gantt
   bar. "Executing Query" duration is measured from the moment Tableau
   dispatches SQL to the connector until the result set fully returns —
   it includes network round-trip time, not just database CPU time, which
   is why the same query can look fast locally and slow over a VPN.
2. For a live connection, VizQL compiles one query per distinct
   query-context change: switching the view from Region to Category isn't
   a filter on cached data, it's a brand-new `GROUP BY Category` query
   sent to the source. An extract instead loads the `.hyper` file's
   columnar store into memory (or memory-maps it) once per session, so a
   Region→Category re-group is answered from local columnar data — no
   round trip, which is the actual mechanism behind "extracts are faster,"
   not a vague performance multiplier.
3. Table calculations execute in a second pass after the query returns:
   the source query produces the aggregated rows (e.g. 3 rows for `Orders`
   grouped by Region — East 2210, West 3750, Central 920), then Tableau's
   local table-calc engine walks that already-returned result set applying
   the addressing/partitioning rule (Level 2 Module 4) to compute things
   like running sum. This is why a table calc never triggers a second
   database query — its cost is CPU time over an in-memory grid, but that
   grid must hold the *entire* partition at once, which is what makes it
   pricier than a plain aggregate as partition size grows.
4. Filter order is enforced by VizQL's query-construction phase, not by
   drag order in the UI: data source filters become `WHERE` clauses baked
   into the generated SQL itself (so the source engine never returns the
   excluded rows at all — for `Category != "Office Supplies"`, the
   generated query never touches rows 1003/1007), context filters are
   materialized as a temp result set the engine treats as the new "table"
   for everything downstream, and dimension/measure filters are applied
   client-side or as a final `WHERE`/`HAVING` against whatever the context
   step produced — which is exactly why only context-filter changes force
   a FIXED LOD to recompute, per Module 1 Section 4.

## Cheat sheet

| Lever | Effect | Cost order (cheapest → priciest per row) |
|---|---|---|
| Data source filter | Reduces rows pulled | Earliest, cheapest overall |
| Extract vs. live | Removes per-query source round-trip | Depends on source, test both |
| LOD FIXED | Computed once, cached | Low |
| Aggregate calc | Once per rendered group | Low–Medium |
| Row-level calc | Once per row | Medium |
| Table calc | Once per mark, needs full partition | High |
| Context filter | Materializes before other filters/LODs | Medium (recomputes on change) |

## Exercise

A workbook applies a data source filter `Category != "Electronics"`
before anything else. Hand-compute: how many `Orders` rows remain (6 —
excluding 1002 and 1005), and what is the new grand total of `Sales`
across the remaining rows? (6880 − 450 − 2200 = **4230**.) Explain why a
row-level calculation like `[Sales]*1.1` now only evaluates 6 times
instead of 8.
