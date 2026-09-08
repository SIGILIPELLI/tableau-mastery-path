# 02 · Connecting to Data Sources

Every Tableau workbook starts on the same screen: the **Connect** pane. This
module covers the connection types you'll use most, the Data Source page
where you shape data before analysis, and the live-vs-extract decision
introduced in Module 1 — using the `Orders` table from Module 1 as the
worked example throughout.

## 1. The Connect pane

1. Opening Tableau Desktop lands you on the **Start screen**, split into a
   **Connect** panel (left) and a list of recent/sample workbooks (right).
2. Under **To a File**, you'll see options including **Microsoft Excel**,
   **Text file** (CSV), **JSON file**, and **PDF file** — the most common
   entry point for a beginner working with an exported spreadsheet.
3. Under **To a Server**, you'll see database and warehouse connectors —
   **Microsoft SQL Server**, **Snowflake**, **Google BigQuery**,
   **PostgreSQL**, and dozens more — each requiring server address and
   credentials rather than a local file path.
4. For this course's `Orders` dataset (Module 1), you'd pick **Microsoft
   Excel** or **Text file** if it were saved as a spreadsheet/CSV — the
   connection type Tableau treats as the simplest, with no server round
   trip needed.

## 2. The Data Source page

After connecting, Tableau opens the **Data Source** page — a canvas for
shaping the data *before* it reaches a worksheet.

1. The top-left lists available **tables/sheets** from your connection; drag
   one onto the canvas to add it.
2. Dragging a second table near the first prompts Tableau to draw a
   **relationship** (the modern default, noeditable double line) or you can
   explicitly build a **join** (inner/left/right/full, a single-line
   connector) — the difference and when to use each is covered in Level 2
   Module 3, since Level 1 uses a single `Orders` table.
3. The bottom half previews the actual rows — use this to sanity-check that
   column names and types look right before building any view. For
   `Orders`, you'd confirm here that `Order Date` is recognized as a
   **Date** type (calendar icon) and `Sales`/`Quantity`/`Profit` as
   **Number** types (# icon), not Text — a common import problem when a
   source file has stray formatting like currency symbols in a numeric
   column.
4. Right-click any column header in the preview to **Rename**, **Change
   Data Type**, or **Hide** — fixing a wrongly-typed column here (e.g. a
   `Sales` column that imported as Text because of a `$` prefix) saves
   having to fix it later inside every worksheet.

## 3. Live vs. Extract, in practice

The **Live/Extract** toggle sits at the top-right of the Data Source page,
and again at the top of the workbook once you're on a worksheet.

1. **Live** stays selected by default for most file and database
   connections. Every filter, sort, or new field triggers a live query
   against the source.
2. To create an **Extract**, click **Extract** (top right) then **Edit...**
   to optionally add filters (e.g. only pull the last two years of orders)
   before Tableau builds the `.hyper` file, or just click the toggle and let
   it pull everything.
3. Once extracted, use **Data > Extract > Refresh** to pull updated source
   data into the existing extract on demand — on Tableau Server/Cloud, this
   refresh can instead be scheduled to run automatically (Level 2 Module 9).
4. Practical rule of thumb for this course's small `Orders` dataset: either
   works identically since the whole table is 8 rows, but in real usage,
   prefer **Extract** for a file-based source you don't expect to change
   during your session — it makes dashboards feel snappier since Tableau
   isn't re-reading the file on every interaction.

## 4. Joining vs. a single flat table

Level 1 uses `Orders` as one flat table specifically so you can focus on
visualization mechanics first. In real datasets, Sales data is often split
across multiple tables (an `Orders` table and a separate `Customers` or
`Products` table linked by an ID) — Tableau calls combining them a
**relationship** or a **join**, both configured on the Data Source page
exactly as described in Section 2. This is covered in depth in Level 2
Module 3, once you've built enough single-table visualizations to appreciate
why splitting data across tables matters.

## How It Actually Works

"Connecting" to data doesn't hand Tableau a static copy — it registers a
**metadata description** (column names, inferred types, and, for
Extract, a materialized copy) that VizQL's query generator reads before it
can build anything:

1. On a **Live** connection, the Data Source page's schema (field names,
   types) is used purely to build correct SQL — every worksheet action still
   round-trips to the real source. Changing `Sales` from Text to Number here
   (as in the Exercise) doesn't change a single byte in the source database;
   it changes the `CAST`/type-coercion Tableau applies to that column when
   it writes the generated query, so `SUM([Sales])` becomes a valid numeric
   aggregate instead of a string-concatenation error.
2. On an **Extract**, connecting triggers Tableau to run one pass over the
   source, apply any type coercions and initial filters from the Data
   Source page, and write the result into a `.hyper` file — a columnar,
   compressed on-disk format. From that point forward, every worksheet query
   is compiled and executed against Hyper locally, not the original source,
   which is why an extract's numbers can drift from a live database until
   you explicitly refresh (Data menu → Extract → Refresh) — you're
   re-running that same import pass, not re-pointing a live pipe.
3. Joining a second table (mentioned at the end of this module, expanded in
   Level 2 Module 3) changes the `FROM`/`JOIN` clause VizQL generates for
   *every* subsequent query, even ones that only reference fields from the
   original table — this is the mechanical reason a badly-chosen join type
   (e.g. an inner join that silently drops unmatched Order IDs) can make
   totals computed later in the course come out lower than a hand
   calculation against `Orders` alone expects.

## Cheat sheet

| Concept | Where / How |
|---|---|
| Connect to a file | Start screen → "To a File" → Excel/Text/JSON/PDF |
| Connect to a database | Start screen → "To a Server" → pick connector |
| Shape data pre-analysis | Data Source page (tables canvas + row preview) |
| Fix a column's type | Right-click header in preview → Change Data Type |
| Toggle Live/Extract | Top-right of Data Source page or worksheet |
| Refresh an extract | Data menu → Extract → Refresh |
| Combine multiple tables | Drag a second table onto Data Source canvas |

## Exercise

Imagine the `Orders` table (Module 1) was imported from a CSV where the
`Sales` column values were formatted as `$1,200.00` and imported as a **Text**
type instead of **Number**. Write down, step by step, exactly which pane you'd
open and which menu commands you'd use to (1) confirm the column is
mistyped, and (2) fix it so `Sales` becomes a proper numeric field ready for
aggregation (SUM) in a worksheet.
