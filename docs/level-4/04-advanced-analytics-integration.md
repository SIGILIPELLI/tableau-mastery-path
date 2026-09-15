---
description: "Tableau + Advanced Analytics Integration (R/Python via Tableau) — Tableau can call out to R or Python for statistical/ML logic beyond its built-in…"
---

# 03 · Tableau + Advanced Analytics Integration (R/Python via Tableau)

Tableau can call out to R or Python for statistical/ML logic beyond its
built-in calculations, via **TabPy** (Python) or an **Rserve** connection.
This module walks through both integration points against `Orders`,
hand-verifying the simplest possible model so the mechanism — not the
statistics — is the focus.

## 1. How the integration works

1. Tableau sends a calculated field's inputs to an external analytics
   service (TabPy for Python, Rserve for R) over a network connection
   configured once (Server/Cloud admin settings, or Desktop's Analytics
   Extension preferences), receives back a computed value per row/partition,
   and displays it like any other calculated field.
2. The calculation itself is written inline using `SCRIPT_REAL`,
   `SCRIPT_STR`, `SCRIPT_BOOL`, or `SCRIPT_INT`, wrapping a Python or R
   snippet as a string, with `_argN` placeholders bound to Tableau
   field arguments in order.

## 2. Worked example: forecasting with a trivial linear fit

1. Using `Orders` aggregated by month (Level 1's dataset spans Jan–Apr
   2024), suppose monthly Sales totals are: Jan 1200+450 = wait — use
   actual monthly rollups: Jan (1001, 1002) = 1200+450=1650; Feb (1003,
   1004) = 60+800=860; Mar (1005,1006,1007) = 2200+950+120=3270; Apr
   (1008) = 1100.
2. A `SCRIPT_REAL` calc could call a Python linear regression
   (`numpy.polyfit`) over these 4 points to project May. This module
   doesn't require running TabPy to understand the mechanism — the key
   verification habit is: before trusting any script-based result, hand
   sanity-check inputs and outputs the same way as any other calc. Here,
   confirm the 4 monthly inputs first sum to the known grand total:
   1650+860+3270+1100 = 6880 ✓ (matches every prior module) — if the
   script's input array doesn't sum to 6880, the aggregation feeding the
   script is wrong before the model logic even runs.

## 3. Argument binding and common mistakes

1. `SCRIPT_REAL("return [x*2 for x in _arg1]", SUM([Sales]))` binds
   `_arg1` to the aggregated `SUM([Sales])` values partitioned per the
   view's granularity — with the view broken out by Region, `_arg1`
   receives `[2210, 3750, 920]` (East, West, Central, in whatever order
   Tableau's partition delivers them) and the script would return
   `[4420, 7500, 1840]` — verify: 2210*2 = 4420, 3750*2 = 7500, 920*2 =
   1840 — a mismatched output length or a value that doesn't correspond
   to any doubled input signals an argument-order or partition-alignment
   bug, one of the most common failure modes with `SCRIPT_*` calcs.
2. `SCRIPT_*` functions are **table calculations** under the hood (Level 3
   Module 5's cost discussion applies) — they run per partition as
   currently laid out in the view, so changing which dimensions are on
   the view changes what `_arg1` contains, even though the script text
   never changed.

## 4. When to use TabPy/Rserve vs. a native LOD/table calc

| Need | Use |
|---|---|
| Standard aggregation, ranking, running total | Native LOD/table calc (Level 2–3) |
| Statistical model (regression, clustering) not expressible in Tableau's calc language | TabPy/Rserve |
| A one-off analysis, not needed live/interactively | Python/R run separately, results imported as a data source |
| Real-time model scoring inside a live dashboard | TabPy/Rserve (accepting the network round-trip cost, Level 3 Module 5) |

## 5. Governance implications

1. A `SCRIPT_*` calculation is a black box to anyone reviewing the
   workbook unless the script is documented — Level 4 Module 2/3's
   governance practices (shared/certified calculations) apply doubly here:
   a certified, well-commented `SCRIPT_REAL` calc should state its exact
   model, inputs, and expected output range (e.g. "returns Sales doubled,
   for input validation only — see Section 3.1") so a future maintainer
   can hand-verify it the same way this module did.

## How It Actually Works

1. TabPy runs as a standalone HTTP service (default port 9004) exposing a
   REST endpoint (`/evaluate`); when VizQL hits a `SCRIPT_REAL` calc, it
   serializes the bound `_argN` arrays as JSON, POSTs them to that
   endpoint along with the script body, and blocks the query pipeline
   until TabPy returns its JSON result — this is a real network round trip
   per query (not a cached local function call), which is exactly why
   Level 3 Module 5's live-vs-extract cost reasoning applies doubly to
   `SCRIPT_*` calcs: every recompute pays TabPy's request latency on top
   of any source query latency.
2. Argument binding is positional, not labeled, because TabPy's protocol
   has no concept of Tableau field names — it only receives arrays of
   values in the order `_arg1, _arg2, ...` were declared in the calc, each
   array aligned to the same partition ordering VizQL used internally for
   that pass. This is the exact mechanism behind the Exercise's bug:
   Tableau's partition order for Category ([Furniture, Electronics, Office
   Supplies] vs. some other order) is an internal implementation detail of
   how the query engine grouped rows, and nothing in the `SCRIPT_REAL`
   protocol carries category *labels* alongside the values — the script
   receives raw positions and must trust they line up with whatever order
   it assumes, which is inherently fragile.
3. Because `SCRIPT_*` functions are compiled as table calculations, they
   inherit table calc's two-pass execution (Level 3 Module 5, Section 3):
   VizQL first runs the aggregate query to produce the per-partition
   `SUM([Sales])` values, then the script call is the local "second pass"
   step, executed once per partition as currently laid out on the view.
   Add or remove a dimension from the view (e.g. break Region down further
   by Category) and the partition boundaries change, so `_arg1` receives a
   differently-shaped array on the very next render — the script's logic
   doesn't change, but its *inputs* silently do, which is why a governance
   note documenting "expected shape of `_arg1`" (Section 5) has to be
   re-verified whenever the view's field layout changes, not just when the
   script text changes.
4. Rserve's protocol differs mechanically (a binary R serialization
   protocol (QAP1) over a TCP socket rather than TabPy's JSON-over-HTTP),
   but the surrounding contract is identical: Tableau blocks on a
   synchronous round trip per script evaluation, arguments arrive
   positionally, and the returned vector must match the partition's row
   count exactly or Tableau raises a calculation error rather than
   silently padding/truncating.

## Cheat sheet

| Function | Returns |
|---|---|
| `SCRIPT_REAL` | Floating-point result per partition |
| `SCRIPT_STR` | String result |
| `SCRIPT_BOOL` | Boolean result |
| `SCRIPT_INT` | Integer result |
| `_argN` | Nth Tableau field/aggregation passed into the script |

## Exercise

A `SCRIPT_REAL` calc computes `_arg1` as `SUM([Sales])` partitioned by
Category, expecting Tableau to pass `[4050, 2650, 180]` (Furniture,
Electronics, Office Supplies, per Level 1's totals). The script instead
receives `[4050, 180, 2650]`. Explain what this indicates (the partition's
category ordering differs from the order assumed when writing the script
— `SCRIPT_*` functions are positional, not labeled, so any code that
assumes a fixed category order, rather than reading dimension labels
alongside `_arg1`, will silently misattribute results) and what a safer
script would do (return results in the same order as the input, and let
Tableau realign them positionally against the same partition — never
hardcode an assumed category order).
