---
description: "Advanced Embedding & Tableau Extensions API Overview — Building on Level 3 Modules 5 and 9, this module covers enterprise-scale embedding and extension…"
---

# 06 · Advanced Embedding & Tableau Extensions API Overview

Building on Level 3 Modules 5 and 9, this module covers enterprise-scale
embedding and extension governance: multi-tenant embedding, extension
allow-listing, and write-back patterns, using `Orders` scenarios.

## 1. Multi-tenant embedding

1. A SaaS product embedding Tableau dashboards for many customer
   organizations needs each customer to see only their own data — the
   embedding equivalent of Level 3 Module 3's RLS, but keyed to a tenant
   identifier rather than (or in addition to) a username.
2. Pattern: extend `RegionAccess`-style entitlement with a `Tenant ID`
   column, and a combined RLS calc:
   `[Tenant ID] = PARAMETER([TenantParam])` alongside the region check —
   the embedding host page sets `TenantParam` via the Embedding API
   (Level 3 Module 9, Section 3's `applyFilterAsync` pattern extends to
   parameters via `parameter.changeValueAsync`) so each customer's
   embedded view is scoped both by tenant and by their own RLS.
3. Verify: Tenant A's East-region user should see the same 2210 total as
   `alice@co.com` did in Level 3 Module 3 — but only if Tenant A's data is
   also isolated; a bug where the tenant filter is applied as a *plain*
   dimension filter rather than a context filter ahead of any FIXED LODs
   could leak a FIXED calc's denominator across tenants (the same pitfall
   as Level 3 Module 1, Section 4, now with tenant isolation at stake
   rather than just a stale total).

## 2. Extension allow-listing and security review

1. Enterprise deployments require explicit allow-listing of extension
   domains (Level 3 Module 5) at the Server/Cloud admin level — an
   ungoverned deployment that allows "any extension" effectively lets any
   dashboard author embed arbitrary third-party JavaScript with access to
   `getUnderlyingDataAsync()` (Level 3 Module 5, Section 2), which for a
   dashboard built on sensitive RLS-gated data is a real exfiltration
   risk: a malicious or careless extension could read all 8 underlying
   `Orders` rows (bypassing the visual-only 3-row Region summary) and
   send them to an external endpoint.
2. Governance response: maintain an explicit allow-list of vetted
   extension domains/manifests (analogous to Level 4 Module 3's
   content-lifecycle discipline, applied to extensions instead of data
   sources), and require security review before any new extension is
   allow-listed, checking exactly what data access (`getSummaryDataAsync`
   vs. `getUnderlyingDataAsync`, per Level 3 Module 5 Section 2) it
   requests.

## 3. Write-back extensions and data integrity

1. Newer Extensions API capabilities allow an extension to write values
   back to a source (e.g. an approval workflow extension writing a
   "Reviewed: Y/N" flag against each `Orders` row) — this introduces a
   new integrity surface: unlike read-only dashboards, a write-back
   extension can *change* the numbers everyone else sees.
2. Guardrail: any write-back extension touching a certified source (Level
   3 Module 8) should itself go through the same certification-adjacent
   review — e.g. requiring the write path to preserve referential
   integrity (an extension that could set `Order ID`, breaking the primary
   key uniqueness `Orders` relies on for every prior module's row counts,
   is a governance failure equivalent to a bad ETL job) and logging every
   write for audit.

## 4. Embedding performance at scale

1. Each embedded `<tableau-viz>` instance on a host page is effectively a
   VizQL Server session — a host page embedding 20 small dashboards (e.g.
   one per Region×Category combination) generates 20 times the VizQL load
   of a single combined dashboard using filters, which is a Module 4
   (Level 4) scaling concern in miniature: prefer fewer, well-designed
   embedded views with interactive filters over many small
   single-purpose embeds when the same data could be shown one way.

## 5. Enterprise embedding checklist

| Concern | Check |
|---|---|
| Tenant isolation | RLS + tenant filter combined, tenant filter applied as context |
| Extension access scope | Summary vs. underlying data, reviewed before allow-listing |
| Write-back integrity | Referential integrity preserved, all writes audited |
| Embedding load | Consolidated views with filters, not many redundant small embeds |

## How It Actually Works

1. `parameter.changeValueAsync` mutates a workbook parameter's stored
   value inside the embedded session the same way typing into a parameter
   control on Server would — VizQL re-evaluates every calculated field
   referencing that parameter (here, the RLS calc's `[Tenant ID] =
   PARAMETER([TenantParam])` comparison) on the next query. Whether that
   comparison behaves as a security boundary or not depends entirely on
   filter placement in the query-evaluation order from Level 3 Module 10:
   as a context filter it's applied *before* any FIXED LOD, so a `{FIXED
   [Region]: SUM([Sales])}` denominator is computed only over the
   tenant-and-region-scoped rows; as a plain dimension filter it's applied
   after, so the FIXED LOD would already have aggregated across every
   tenant's rows before the visible filter ever trims the display — the
   number shown could be visually scoped to Tenant A while its underlying
   FIXED denominator silently includes Tenant B's Sales.
2. Extensions run inside a sandboxed iframe and talk to the host dashboard
   through the Extensions API's message-passing layer, exactly like the
   Embedding API (Level 3 Module 9) — `getSummaryDataAsync()` requests the
   *already-aggregated, already-RLS-filtered* result set VizQL rendered
   for the current view (e.g. 3 rows, one per visible Region), while
   `getUnderlyingDataAsync()` requests the row-level data the view's marks
   are built from, before final visual aggregation — critically, both
   still pass through RLS (Level 3 Module 3), because RLS is enforced at
   the query layer, not the rendering layer, so `getUnderlyingDataAsync()`
   cannot see rows outside the current viewer's RLS scope. The real risk
   in Section 2 and the Exercise isn't an RLS bypass — it's requesting
   *unnecessarily granular* access within the viewer's own already-visible
   scope (all 8 rows a permitted viewer could see, rather than the 3
   region subtotals the chart actually needs), which is disproportionate
   access even though it's not a security hole in the strict RLS sense.
3. Write-back extensions use the Extensions API's data-source write
   methods (or a custom backend the extension calls out to) — Tableau
   itself does not natively validate referential integrity on write; an
   extension writing an "Order ID" value it invents can create duplicate
   or non-existent keys unless the write path enforces the constraint
   itself, which is exactly why Section 3's guardrail requires the write
   path's own validation logic to be reviewed, not just the extension's
   read-side data access.
4. Embedding load scales because each `<tableau-viz>` custom element opens
   its own iframe and its own VizQL Server session token (Level 3 Module 9)
   — 20 embedded single-purpose views are 20 independent sessions with 20
   independent query-compilation and rendering cost, whereas one combined
   dashboard with interactive Region/Category filters is one session whose
   filter changes reuse the same already-open session rather than spinning
   up a new one per view.

## Exercise

A newly proposed extension requests `getUnderlyingDataAsync()` access on
a dashboard built on the RLS-gated certified `Orders` source, but its
stated purpose ("show a bar chart with totals by Region") only needs
`getSummaryDataAsync()`. Using Section 2's reasoning, explain why this
mismatch should block allow-listing until resolved (requesting row-level
underlying data for a purely aggregate use case is disproportionate
access — it would let the extension see all 8 individual orders,
including for regions the current viewer's RLS should otherwise restrict
via the dashboard's visual layer, undermining Level 3 Module 3's
entitlement model even if the extension's UI itself only displays
aggregates).
