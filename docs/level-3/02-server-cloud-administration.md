# 02 · Tableau Server/Cloud Administration Basics

This module shifts from building content to administering the platform it
lives on. Concepts are illustrated using the `Orders` workbook and
`Northwind Analytics` project published in Level 2 Module 9/10.

## 1. Server topology basics

1. A Tableau Server installation runs several **processes** across one or
   more nodes: **Gateway** (routes incoming requests), **VizQL Server**
   (renders views), **Backgrounder** (runs extract refreshes/subscriptions),
   **Repository** (Postgres metadata store), and **Data Server** (serves
   published data sources like the cross-database `Orders`+`Targets` join).
2. Tableau Cloud is the fully-managed SaaS equivalent — Tableau hosts and
   scales these same process roles; an admin manages **sites**, **users**,
   and **content**, not servers.

## 2. Sites, projects, and the content hierarchy

1. A **Server** (or Cloud pod) contains one or more **Sites** (isolated
   tenants — e.g. "Sales" vs. "Finance" sites sharing one Server
   installation but never sharing content or users by default).
2. Within a site, **Projects** are folders with their own permission
   templates — the `Orders` workbook from Level 2 would live in a
   "Northwind Analytics" project, itself possibly nested under a top-level
   "Sales Reporting" project.
3. Nested projects can **lock** permissions so sub-projects inherit rather
   than override — appropriate when every dashboard under "Sales
   Reporting" should share the same viewer group.

## 3. Users, groups, and site roles

1. **Site roles** form a ceiling on what a user can do regardless of
   individual content permissions: Viewer < Explorer < Creator (Creator can
   publish new workbooks and data sources; Explorer can interact with and
   build from published sources but not publish new connections; Viewer
   can only view/interact with what's published).
2. Assign the `Orders` dashboard's intended sales-manager audience the
   **Viewer** role plus explicit "View" and "Download Image" content
   permissions — but not "Download Full Data," per the restriction set in
   Level 2 Module 9.
3. Use **Groups** (e.g. "Sales-Managers") rather than per-user permission
   grants — adding a new manager later means adding them to the group
   once, not re-granting permissions on every workbook individually.

## 4. Backgrounder and scheduled tasks

1. Extract refresh schedules (Level 2 Module 9) and subscription emails
   both run as **Backgrounder** jobs — an admin monitors these under
   **Status** → **Background Tasks**, checking for failed refreshes (e.g.
   a schedule that failed because the source `Orders` file was moved or
   renamed).
2. A failed nightly refresh of `Orders` would leave the published
   dashboard's Region totals stuck at the last successful values (2210 /
   3750 / 920) even as the real source data changes — admins should alert
   on repeated refresh failures, not just single transient ones.

## 5. Licensing model overview

1. Tableau licenses per **named user** (each login consumes one license of
   its assigned role) or per **core** (server capacity licensed
   regardless of user count) — small deployments typically use named-user
   licensing; large, unpredictable-usage deployments often use core-based.
2. Preview in depth in Level 4 Module 7 (Cost & License Management).

## How It Actually Works

Understanding which process handles which stage of a request explains why
certain failures show up where they do, and why the `Orders` dashboard's
totals sometimes lag rather than error out:

1. When a browser opens the `Orders` dashboard, **Gateway** routes the
   HTTP request to an available **VizQL Server** process, which loads the
   workbook definition, generates the same style of `GROUP BY` query this
   course has built up (Level 1 Module 1), and — for a published data
   source rather than an embedded one — routes that query through **Data
   Server**, which manages connection pooling and caching for shared
   sources like the cross-database `Orders`+`Targets` join (Level 2 Module
   10). None of these processes touch the extract's *contents* directly;
   they only run queries against whatever `.hyper` file currently exists
   on disk.
2. **Backgrounder** is the only process that actually rewrites that
   `.hyper` file — a scheduled refresh job is a Backgrounder task that
   re-runs the same extract-creation pass from Level 1 Module 2 (pull rows,
   apply extract filters/aggregation from Level 2 Module 8, write a new
   `.hyper`). This is mechanically why a failed Backgrounder job leaves
   Region totals frozen at 2210/3750/920: VizQL Server keeps querying the
   old, still-valid `.hyper` file it has no reason to know is stale — no
   process actively "pushes" fresh data to VizQL Server, it only pulls
   whatever the extract currently contains.
3. **Site and Project isolation** are enforced at the metadata layer
   (**Repository**, the Postgres store) before a query ever reaches VizQL
   Server: a request for a workbook in a Site the requesting user's
   session isn't authorized for is rejected by permission checks against
   Repository records, never by VizQL Server discovering it can't run the
   query — this is why permission and licensing errors surface as
   access-denied pages, not as query failures.
4. **Named-user vs. core licensing** (Section 5) changes what Gateway
   checks at login/session-creation time, not what VizQL Server computes —
   a named-user deployment checks the logging-in identity against a
   consumed-license table in Repository, while core licensing has no such
   per-user check at all, only a total concurrent-load ceiling enforced
   across all the server's processes.

## Cheat sheet

| Concept | Role |
|---|---|
| Gateway | Routes requests |
| VizQL Server | Renders views |
| Backgrounder | Runs refreshes/subscriptions |
| Repository | Postgres metadata store |
| Site | Isolated tenant within a Server/Cloud pod |
| Project | Folder + permission template |
| Site role | Viewer / Explorer / Creator ceiling |
| Group | Bulk permission management unit |

## Exercise

Design a site-role and group assignment for three people: (1) the analyst
who built the `Orders` dashboard and needs to publish updates, (2) a sales
manager who only views it, (3) an ops person who needs to fix a failed
extract refresh but not edit the dashboard's design. State each person's
site role and which group(s), if any, they'd belong to.
