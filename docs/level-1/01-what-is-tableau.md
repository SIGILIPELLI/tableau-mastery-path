---
description: "What Is Tableau? — Tableau is a business intelligence (BI) and data visualization platform: you connect it to data (spreadsheets, databases, cloud…"
---

# 01 · What Is Tableau?

Tableau is a **business intelligence (BI) and data visualization platform**:
you connect it to data (spreadsheets, databases, cloud warehouses), and it
lets you build interactive charts and dashboards by dragging fields onto a
canvas rather than writing charting code. This module maps out the product
family so the rest of the course makes sense, and introduces the small
fictional dataset every later Level 1 lesson builds on.

## 1. The Tableau product family

Tableau is not one product — it's a family of related tools, and knowing
which one a task belongs to avoids a lot of confusion later.

1. **Tableau Desktop** — the authoring application. You install it on your
   machine, connect to data, and build worksheets and dashboards ("workbooks",
   saved as `.twb` or `.twbx` files). This is what you'll use for nearly all
   of Level 1 and Level 2.
2. **Tableau Public** — a free version of Desktop-style authoring, with one
   restriction: every workbook you save is published publicly to
   `public.tableau.com` — there is no private/local save. Great for
   learning and for a public portfolio; not for confidential company data.
3. **Tableau Server** — a self-hosted (on your own or your company's
   servers) platform for publishing, sharing, and scheduling refreshes of
   workbooks so other people can view and interact with them in a browser
   without owning a Desktop license.
4. **Tableau Cloud** (formerly Tableau Online) — the same publishing/sharing
   experience as Server, but hosted by Salesforce/Tableau instead of your
   own infrastructure — no servers to maintain.
5. **Tableau Prep** — a separate application for cleaning and reshaping data
   *before* it reaches a workbook: renaming/splitting columns, joining
   tables, fixing inconsistent values. Covered starting in Level 2.

!!! info "Which one do I need for this course?"
    Tableau Desktop (or Tableau Public, if you don't have a Desktop license)
    for everything through Level 2. Server/Cloud concepts are covered
    conceptually from Level 2 Module 9 onward and in depth in Level 3 — you
    don't need your own server to follow those lessons, since the workflows
    and admin concepts are explained step-by-step with what you'd see in the
    interface.

## 2. How Tableau "thinks" about data

Tableau's core idea is the **worksheet**: one visualization, built by
dragging fields from the **Data pane** (left side of the screen, listing
every column in your connected data) onto **shelves** — labeled drop targets
like Columns, Rows, Color, Size, Label, and Filters — arranged around the
canvas. The **Marks card** controls the shape of the mark itself (bar, line,
circle, etc.) and lets you drop fields onto Color/Size/Label/Detail/Tooltip
to encode more information into each mark.

Tableau splits every field into one of two roles:

- **Dimensions** — categorical or descriptive data (Region, Product,
  Customer Name, Order Date). Dimensions slice your view into groups.
- **Measures** — numeric data meant to be aggregated (Sales, Profit,
  Quantity). Measures get summarized (SUM, AVG, COUNT, etc.) by default.

Tableau also colors fields **blue** (discrete) or **green** (continuous) in
the Data pane and on shelves — a dimension is usually discrete/blue, a
measure is usually continuous/green, but either can be switched (right-click
a field on a shelf → **Discrete** or **Continuous**), which changes how it
renders (headers/labels vs. an axis).

## 3. The running dataset for this course

Every Level 1 lesson builds against the same small fictional dataset: a
retail chain called **Northwind Retail**, tracking orders across three
regions. You don't need to load this into Tableau to follow along — the
lessons walk through the exact clicks and show you the resulting numbers —
but writing it down now means every calculated field and chart result later
is something you can check by hand.

**Table: `Orders`**

| Order ID | Order Date | Region | Category | Product | Sales | Quantity | Profit |
|---|---|---|---|---|---|---|---|
| 1001 | 2024-01-05 | East | Furniture | Desk | 1200 | 2 | 180 |
| 1002 | 2024-01-12 | West | Electronics | Monitor | 450 | 3 | 90 |
| 1003 | 2024-02-03 | East | Office Supplies | Paper | 60 | 10 | 18 |
| 1004 | 2024-02-20 | Central | Furniture | Chair | 800 | 4 | 96 |
| 1005 | 2024-03-01 | West | Electronics | Laptop | 2200 | 2 | 330 |
| 1006 | 2024-03-15 | East | Furniture | Bookcase | 950 | 1 | -40 |
| 1007 | 2024-03-22 | Central | Office Supplies | Binders | 120 | 8 | 42 |
| 1008 | 2024-04-02 | West | Furniture | Desk | 1100 | 1 | 150 |

Eight orders, four columns of dimensions (Region, Category, Product, Order
Date) and three measures (Sales, Quantity, Profit) — small enough to total
by hand, which is exactly the point: when a later lesson says "SUM(Sales)
for the East region is 2210", you can verify it (1200 + 60 + 950 = 2210).

## 4. Live connection vs. extract — a first look

When you connect Tableau to a data source, you choose between:

- **Live** — every action in Tableau sends a fresh query to the underlying
  data source. You always see current data, but performance depends on that
  source's speed.
- **Extract** — Tableau pulls a compressed, optimized snapshot (a `.hyper`
  file) of the data into the workbook itself. Fast and portable, but the
  data is a point-in-time snapshot until you manually or automatically
  refresh it.

This choice is made the moment you connect to data — covered in full in
Module 2.

## How It Actually Works

Every drag onto a shelf is, under the hood, building an abstract query
Tableau's **VizQL** engine will translate into a real query against whatever
you're connected to (SQL for a database/extract, MDX for an OLAP cube). You
never see this query directly in Level 1, but knowing it exists explains a
lot of later behavior:

1. Dragging **Region** to Rows and **SUM(Sales)** to Columns produces
   roughly the query pattern `SELECT Region, SUM(Sales) FROM Orders GROUP BY
   Region ORDER BY Region` — every dimension on a shelf becomes a `GROUP BY`
   term, every measure becomes an aggregate in the `SELECT` list. This is
   why adding a second dimension (say, Category) doesn't just add a column —
   it changes the `GROUP BY` to `Region, Category`, which subdivides every
   existing group into smaller ones (Furniture-East, Electronics-East, etc.)
   rather than adding new independent rows.
2. **Live vs. Extract** (Section 4) changes *where* this generated SQL runs,
   not *whether* it exists. On Live, that `SELECT ... GROUP BY` is sent to
   your actual database every time you touch a shelf. On Extract, the same
   query runs against Tableau's own embedded **Hyper** engine — a columnar
   analytical database bundled into the `.hyper` file — which is why extracts
   are often faster: Hyper stores each column contiguously and compressed,
   so `SUM(Sales)` only has to scan the Sales column, not full rows.
3. This is also the mechanical reason dimensions and measures behave
   differently: a dimension is, by construction, something VizQL can put in
   a `GROUP BY`/axis-header list, while a measure is something it can only
   put inside an aggregate function. Manually switching a field to
   Discrete/Continuous (Section 2) is really telling VizQL whether to treat
   its values as `GROUP BY` buckets (discrete, headers) or as a numeric axis
   domain (continuous, a line/scale) — the same underlying column, two
   different query shapes.
4. Hand-verification tie-in: when Module 3 has you check that the East bar
   reads 2210, what you're really confirming is that VizQL's generated
   `GROUP BY Region` query summed the right three rows (1200 + 60 + 950)
   from the `Orders` table above — the same arithmetic a `SUM(Sales) WHERE
   Region = 'East'` SQL query would perform against a real database.

## Cheat sheet

| Term | Meaning |
|---|---|
| Tableau Desktop | Authoring app — builds workbooks (.twb/.twbx) |
| Tableau Public | Free authoring, but every save publishes publicly |
| Tableau Server | Self-hosted publishing/sharing platform |
| Tableau Cloud | Vendor-hosted publishing/sharing platform |
| Tableau Prep | Separate app for cleaning/reshaping data pre-analysis |
| Dimension | Categorical field (slices the view); default blue/discrete |
| Measure | Numeric field (gets aggregated); default green/continuous |
| Data pane | Left-side list of every field in the connected data |
| Shelf | Drop target for fields (Columns, Rows, Filters, etc.) |
| Marks card | Controls mark type + Color/Size/Label/Detail/Tooltip |
| Live connection | Queries the source fresh every time |
| Extract | Snapshot of data saved into the workbook (.hyper) |

## Exercise

Without opening Tableau yet: using the `Orders` table above, compute by hand
(1) total Sales for each Region, (2) total Profit for the Furniture category
across all regions, and (3) which single order has the worst Profit. Write
down your three answers — Module 3 will have you build a Tableau view that
reproduces the first of these, and you'll be able to check your worksheet
against the number you just calculated.
