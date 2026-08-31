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
