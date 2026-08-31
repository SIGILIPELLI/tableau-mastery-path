# 08 · Performance Optimization Basics

`Orders` has only 8 rows, so nothing here will actually feel slow — but the
diagnostic habits taught in this module apply identically at 8 rows or 8
million. We'll reason about *why* each technique helps using this dataset's
known numbers.

## 1. The Performance Recording tool

1. **Help → Settings and Performance → Start Performance Recording**, use
   the workbook, then **Stop Performance Recording** — Tableau opens a
   generated dashboard showing a timeline of every query, layout, and
   rendering event with its duration.
2. Events worth watching for real workbooks: "Executing Query" bars far
   longer than others usually mean the data source query itself (not
   Tableau's rendering) is the bottleneck — the fix is almost always on the
   data/extract side, not the worksheet design.

## 2. Extracts vs. live for performance

1. An **extract** (`.hyper` file) pre-aggregates and compresses data into
   Tableau's own columnar engine — for `Orders`, extracting means every
   view (Region totals, Category totals) is served from a local
   already-optimized copy rather than re-querying the source file.
2. **Extract with aggregation**: Data Source page → Extract → Edit →
   check "Aggregate data for visible dimensions" — for a workbook that
   only ever needs Region/Category-level totals (2210/3750/920 and
   4050/2650/180), this option pre-collapses the 8 row-level records into
   the 3+3 aggregate rows actually used, shrinking the extract further.

## 3. Filters: extract filters vs. context filters vs. quick filters

1. An **extract filter** (Data Source page → Extract → Filters) removes
   rows *before* they ever enter the `.hyper` file — e.g. filtering to
   `Order Date >= 2024-02-01` at extract time would drop orders 1001 and
   1002 (Jan) permanently from that extract, reducing every downstream
   query's scan size.
2. A **context filter** (right-click a filter pill → Add to Context)
   forces that filter to run first, before other filters/table calcs,
   which both fixes certain table-calc-ordering bugs (Level 2 Module 4)
   and, for a large dataset, lets Tableau cache the reduced result set
   once instead of recomputing it for every other filter change.
3. **Quick filters** (a filter card left on the dashboard for viewers) are
   the most expensive per-interaction — each click re-queries — so keep
   the number of quick filters and their cardinality (distinct value
   count) low; `Region` (3 distinct values) is a cheap quick filter, a
   hypothetical `Order ID` (8, or millions in a real dataset) is not.

## 4. Reducing marks and calculation complexity

1. A view rendering one mark per `Order ID` (8 marks here) is cheap; the
   same *pattern* at real scale — one mark per row in a million-row table
   — is a common performance mistake. Prefer aggregating to Region/Category
   (3–4 marks) whenever the analysis question doesn't require row-level
   detail.
2. Nested LOD expressions (Level 2 Module 1) recompute per underlying row
   before aggregating — a `{FIXED [Region]: SUM([Sales])}` over 8 rows is
   instant, but the same construct over a large fact table forces a full
   table scan per LOD, so prefer a pre-aggregated extract (Section 2) when
   an LOD's underlying grain is much finer than the view needs.

## 5. Verifying an optimization actually helped

1. Before/after check: extract-filter the workbook to `Order Date >=
   2024-02-01` (Section 3.1) and confirm `SUM(Sales)` now reads 60 + 800 +
   2200 + 950 + 120 + 1100 = **5230** (orders 1003–1008), not 6880 — this
   confirms the filter genuinely reduced the row set rather than just
   hiding rows visually.
2. Any optimization step should be paired with exactly this kind of
   before/after total check — performance work that silently changes your
   numbers is a correctness bug, not a speed win.

## Cheat sheet

| Technique | Effect |
|---|---|
| Extract (`.hyper`) | Local, pre-optimized copy vs. live re-query |
| Extract with aggregation | Collapses to only the dimensions used |
| Extract filter | Drops rows before the extract is built |
| Context filter | Runs first; can be cached; fixes calc-order bugs |
| Fewer marks / less granularity | Less rendering + query work |
| Performance Recording | Diagnoses where time is actually spent |

## Exercise

Using an extract filter of `Order Date >= 2024-03-01`, list which Order IDs
remain (1005, 1006, 1007, 1008) and hand-verify the new `SUM(Sales)` total
is 2200+950+120+1100 = 4370.
