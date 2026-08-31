# 09 · Publishing to Tableau Public/Server

Building a workbook is half the job — this module covers getting the
`Orders` dashboard (Level 1 Module 5, refined through Level 2) in front of
an audience via Tableau Public or Tableau Server/Cloud.

## 1. Tableau Public vs. Server/Cloud

| | Tableau Public | Tableau Server / Cloud |
|---|---|---|
| Audience | Anyone with the link (fully public) | Licensed users in your org |
| Data visibility | Underlying data is downloadable by viewers | Data stays behind auth |
| Cost | Free | Licensed (per-user or per-core) |
| Use case | Portfolio pieces, open journalism data | Internal BI, sensitive data |

For the `Orders` dataset (sample data, no confidentiality concerns),
Tableau Public is appropriate; a real company's actual sales figures would
require Server/Cloud instead.

## 2. Publishing to Tableau Public

1. **Server → Tableau Public → Save to Tableau Public**, sign in, choose a
   workbook name (e.g. "Northwind Retail — Regional Sales").
2. Public requires the data source be an **extract** (not live) — since
   `Orders` is a small flat file, this is a non-issue; Tableau converts it
   automatically if needed.
3. After publishing, the workbook gets a public profile URL; embed code
   (an `<iframe>` snippet) is available from the "Share" button on the
   published view for pasting into a blog or portfolio site.

## 3. Publishing to Tableau Server/Cloud

1. **Server → Publish Workbook**, select the target **Project** (a
   folder-like container for access control) and **data source publish
   option**: embed the data source in the workbook, or publish it
   separately as a reusable, centrally-refreshed data source.
2. Publishing `Orders` as a separate data source (rather than embedded)
   means a second workbook (e.g. a Category-focused dashboard) can connect
   to the same already-cleaned `Orders` source without re-doing Prep work
   — the standard pattern once more than one dashboard needs the same data.
3. Set **permissions** at the project or workbook level (View, Interact,
   Download Full Data, Web Edit) — for a workbook containing real Sales
   figures, restrict "Download Full Data" for viewers who should only see
   aggregate dashboards, not export the 8-row source.

## 4. Extract refresh schedules

1. Once published to Server/Cloud, an extract-based data source can be
   attached to a **refresh schedule** (e.g. nightly at 2am) so the
   published `.hyper` file stays current without manual re-publishing.
2. For `Orders`, imagine new rows are appended to the source CSV weekly —
   a weekly Sunday-night schedule keeps the published dashboard's totals
   (currently 6880) accurate going forward without anyone touching Server.

## 5. Subscriptions and alerts

1. **Subscribe Others** (or self-subscribe) sends a scheduled email
   snapshot of a published view — e.g. a Monday-morning email of the
   Region Sales bar chart (2210/3750/920) to a sales manager.
2. **Data-Driven Alerts**: configure a threshold on a measure — e.g. alert
   if any Region's Sales drops below 900 (Central, at 920, is closest to
   this threshold in the current data) — Tableau emails automatically the
   next time the underlying data crosses that line.

## 6. Verifying a publish succeeded

1. After publishing, open the **published** URL (not the local `.twbx`)
   in a browser and confirm the Region totals still read 2210 / 3750 /
   920 — a mismatch usually means the publish used a stale extract or an
   embedded copy of the data source diverged from the live source.
2. Check permissions by testing as a lower-privilege test account (or
   Server's "View As" feature) to confirm a viewer-level user cannot
   download the underlying 8-row table if that was intentionally
   restricted.

## Cheat sheet

| Task | Where |
|---|---|
| Publish (public, free) | Server → Tableau Public → Save to Tableau Public |
| Publish (org, licensed) | Server → Publish Workbook |
| Reusable data source | Publish data source separately from workbook |
| Set access | Project/workbook Permissions dialog |
| Keep data current | Extract refresh schedule |
| Recurring snapshot | Subscribe (self or others) |
| Threshold notification | Data-Driven Alert |

## Exercise

Design a data-driven alert rule for the `Orders` dashboard that would have
fired historically: using the known Region totals (2210, 3750, 920), pick a
threshold that only Central would cross, state the rule in plain language,
and explain who should receive it and why.
