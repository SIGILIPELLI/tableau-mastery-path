---
description: "Tableau Prep Basics — Tableau Prep Builder is a separate application for cleaning and reshaping data before it reaches Tableau Desktop. This module walks…"
---

# 06 · Tableau Prep Basics

Tableau Prep Builder is a separate application for cleaning and reshaping
data *before* it reaches Tableau Desktop. This module walks a Prep flow
using a messier version of `Orders` with the problems Prep is designed to
fix.

## 1. The messy source

Imagine `orders_raw.csv` exported with these issues, using the same 8
underlying orders:

| Order ID | Order Date | Region | Sales |
|---|---|---|---|
| 1001 | 01/05/2024 | east | $1,200.00 |
| 1002 | 01/12/2024 | WEST | $450.00 |
| 1003 | 02/03/2024 | East | $60.00 |
| ... | ... | ... | ... |

Problems: inconsistent Region casing (`east`, `WEST`, `East`), Sales stored
as text with a `$` and comma, and Order Date as a US-format string rather
than a real date type.

## 2. Flow structure: Input → Clean → Output

1. A Prep **flow** is a left-to-right pipeline of steps, each a node:
   **Input** (connect to `orders_raw.csv`) → one or more **Clean/Add
   Column** steps → **Output** (write a cleaned `.hyper` extract or publish
   to Server).
2. Each step shows a live **data grid** plus a **profile pane** (histogram
   per column) so you see the effect of a transformation immediately,
   rather than guessing.

## 3. Cleaning: casing and type fixes

1. Right-click the `Region` column → **Clean** → **Change Case** → propose
   `TITLE CASE`, turning `east`/`WEST`/`East` all into `East`/`West`/`East`
   — after this, grouping by Region correctly yields 3 distinct values
   instead of 5 (`east`, `East`, `WEST`, `West`, `Central` if Central had
   similarly varied).
2. Right-click `Sales` → **Clean** → change data type from String to
   Number (Decimal); Prep auto-strips the `$` and thousands comma during
   the conversion, turning `$1,200.00` into the number `1200`.
3. Right-click `Order Date` → change type from String to Date, specifying
   the `MM/DD/YYYY` input pattern so `01/05/2024` parses to a real date
   rather than staying text (which would sort alphabetically, not
   chronologically).

## 4. Grouping similar values (fuzzy match)

1. Prep's **Group Values** → **Common typos/character order** helps catch
   near-duplicates a simple case-fix wouldn't — e.g. `"Office Supplies"`
   vs. `"Office Supply"` vs. `"OfficeSupplies "` (trailing space)
   collapsing to one canonical `Office Supplies` group.
2. Verify after grouping: `COUNTD([Category])` should read exactly **3**
   (Furniture, Electronics, Office Supplies) — if it reads higher, an
   unmerged variant is still present and needs another grouping pass.

## 5. Aggregate and Pivot steps

1. An **Aggregate** step can pre-summarize before output — e.g. group by
   Region, `SUM(Sales)`: East 2210, West 3750, Central 920, matching the
   hand totals used throughout this course.
2. A **Pivot** step reshapes wide-to-long or long-to-wide — e.g. if a
   source system exported one column per month (`Jan_Sales`, `Feb_Sales`,
   `Mar_Sales`, `Apr_Sales`: 1650, 860, 3270, 1100), Pivot → Columns to Rows
   turns those 4 columns into 2 (`Month`, `Sales`), producing 4 rows per
   original row — necessary before Tableau Desktop can treat Month as a
   proper dimension.

## 6. The Output step and refresh

1. **Output** step: choose **Save to file** (`.hyper` extract) for local
   use, or **Publish as data source** to push the cleaned flow directly to
   Tableau Server/Cloud where Desktop can connect to it.
2. Flows can be scheduled to re-run automatically on Tableau Server (via
   **Prep Conductor**, Level 3), keeping the cleaned output current without
   manually re-running Prep Builder each time the source CSV updates.

## How It Actually Works

Prep Builder's flow is a **directed pipeline of transformation nodes**,
executed once at run time (interactively for the preview, and again in full
at Output) rather than a set of live formulas re-evaluated forever — this
execution model explains both the "fix casing before grouping" ordering and
Section 6's whitespace-trim ordering question:

1. Each node in the flow (Clean, Group, Aggregate, Pivot) runs as a
   **discrete transformation pass over the full row set output by the
   previous node** — conceptually equivalent to a sequence of `SELECT ...`
   statements chained via CTEs, each one materializing (in the preview
   grid) the exact rows the next step will see. This is why a Change Case
   step must sit *before* a Group Values step in the flow: Group Values
   operates on whatever string values are already present when it runs, so
   feeding it un-normalized casing (`east`/`WEST`/`East`) means it has to
   independently discover and merge three variants instead of the one
   canonical `East` a prior Change Case step would have already produced.
2. **Type conversion is a per-value parse operation**, not a bulk
   reinterpretation — converting `Sales` from String to Number applies a
   parse function per cell that strips currency symbols/commas as part of
   the coercion, but a stray leading/trailing space (Exercise's `" 120 "`)
   can make that parse fail or silently produce a null/error depending on
   the connector, because whitespace isn't part of the currency-stripping
   pattern the type-conversion step applies. Trimming whitespace with a
   Clean step *before* the type conversion guarantees the converter only
   ever sees a clean numeric-looking string (`"120"`), which is why the
   canonical fix order is trim → then convert, not convert → then trim.
3. **Aggregate and Pivot steps change the row count contract** for every
   downstream node: an Aggregate step collapsing 8 Order rows to 3 Region
   rows means any later step operating "per row" now operates on Region
   granularity, not Order granularity — the same kind of grain-shift
   Level 2 Module 1's LOD expressions handle inside Tableau Desktop, just
   performed as a physical, materialized step inside Prep instead of a
   virtual subquery inside VizQL.

## Cheat sheet

| Task | Prep step |
|---|---|
| Fix inconsistent casing | Clean → Change Case |
| Strip currency symbols, fix type | Clean → change data type |
| Parse a text date | Clean → change type to Date + pattern |
| Merge near-duplicate values | Clean → Group Values |
| Pre-summarize rows | Aggregate step |
| Reshape wide→long | Pivot → Columns to Rows |
| Write cleaned result | Output → Save to file / Publish |

## Exercise

Given a hypothetical `orders_raw.csv` where Central's Sales values were
exported as `"800"` and `" 120 "` (note the stray leading/trailing spaces
on the second), describe the Prep steps needed so that after cleaning,
`SUM(Sales)` for Central correctly reads 920, and explain what would go
wrong if the type conversion happened before the whitespace trim.
