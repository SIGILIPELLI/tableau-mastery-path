# 01 · Enterprise Tableau Architecture

Level 4 shifts from building/administering one deployment to designing
one across an organization. This module lays out reference architectures,
sizing considerations, and disaster-recovery patterns, grounding examples
in the `Northwind Retail` data used throughout the course.

## 1. Reference architecture: single-site vs. multi-site

1. A **single-site** Tableau Server deployment puts every team's content
   (Sales' `Orders` dashboards, Finance's reports, etc.) under one Site,
   relying on Projects and permissions (Level 3 Module 2) to separate
   audiences.
2. A **multi-site** deployment isolates tenants completely — e.g. a
   "Sales" site holding the `Northwind Analytics` project from Level 2,
   and a separate "Finance" site with no shared users or content by
   default — appropriate when regulatory or organizational boundaries
   require hard separation (different data residency rules, or
   subsidiaries that shouldn't see each other's content at all).
3. Trade-off: multi-site avoids any cross-team leakage risk but duplicates
   admin overhead (server processes, license pools' scoping if
   applicable) — most enterprises start single-site with strong project
   permissions, and split into sites only for a specific isolation
   requirement.

## 2. Node topology and process distribution

1. A minimal production Server deployment separates roles across nodes:
   e.g. one node running Gateway + VizQL Server (handling viewer traffic),
   a second running Backgrounder (running the nightly extract refresh for
   `Orders`, per Level 3 Module 4's incremental refresh pattern) and Data
   Server, and a third running the Repository (Postgres metadata).
2. Sizing decision: Backgrounder capacity should scale with the number and
   size of scheduled extract refreshes, not viewer count — a deployment
   with a handful of viewers but hundreds of nightly extract jobs (each
   like the `Orders` incremental refresh from Level 3 Module 4) needs more
   Backgrounder capacity than viewer-facing capacity, and vice versa for a
   read-heavy, refresh-light deployment.

## 3. High availability and disaster recovery

1. **High availability (HA)** adds redundant nodes for each critical
   process so a single node failure doesn't take the whole Server down —
   e.g. two Gateway nodes behind a load balancer so viewer traffic to the
   `Orders` dashboard continues if one Gateway node fails.
2. **Disaster recovery (DR)** is a separate concern: a secondary Server
   deployment (often in a different data center/region) that can be
   promoted if the primary deployment is lost entirely — this typically
   relies on a Backup/Restore (`tsm maintenance backup`) taken on a
   schedule and shipped to the DR site, meaning DR recovery point is
   bounded by backup frequency (a nightly backup means up to 24 hours of
   content changes could be lost in a DR failover — e.g. a same-day fix to
   the `Orders` RLS entitlement table, Level 3 Module 3, made after the
   last backup would need to be reapplied manually after a DR failover).

## 4. Capacity planning inputs

1. Four inputs drive sizing: concurrent viewer count, extract refresh
   volume/frequency (Backgrounder load), workbook complexity (calc-heavy
   dashboards like Level 3 Module 10's project cost more per view than a
   simple bar chart), and data volume per extract (an `Orders`-scale
   8-row extract is negligible; a production fact table at millions of
   rows changes memory/CPU sizing substantially).
2. A common mistake is sizing only for viewer count and ignoring
   Backgrounder load — a deployment that looks "right-sized" for 500
   concurrent viewers can still fail its nightly refresh window if 200
   scheduled extracts (each independently no larger than `Orders`, but
   collectively large) are all scheduled to run in the same 2-hour
   window.

## 5. Choosing Server vs. Cloud at the architecture level

| Factor | Favors Server (self-hosted) | Favors Cloud (SaaS) |
|---|---|---|
| Data residency/on-prem source proximity | Yes — data stays behind firewall | Requires bridge/gateway for on-prem sources |
| Ops team capacity | Needs dedicated admin capacity | Salesforce manages infra |
| Custom node topology (Section 2) | Full control | Managed, less topology control |
| Scaling speed | Manual node provisioning | Elastic, managed by vendor |

## How It Actually Works

1. Sites are a hard tenancy boundary implemented at the metadata layer:
   every content object (workbook, data source, project) in the
   Repository (Postgres) carries a `site_id` foreign key, and every query
   Server executes is scoped by the caller's site membership — a Sales
   site user's session simply cannot resolve a Finance site's content ID,
   because the query never includes rows outside that `site_id`. A
   single-site deployment with project permissions instead uses a
   `project_id`/permission-grant model within the same site — a much
   finer-grained but *not* hard-isolated boundary, since a site admin (and
   anyone Tableau's permission model grants cross-project visibility to)
   can still see across projects, which is the concrete mechanism behind
   "multi-site avoids cross-team leakage risk" in Section 1.
2. Node roles are literal Linux processes/services configured via `tsm
   topology` — Gateway (a reverse proxy routing incoming requests),
   VizQL Server (compiles and runs queries, renders views), Backgrounder
   (pulls extract-refresh and subscription jobs from a queue), and
   Repository (Postgres, holding all metadata). Assigning a node to
   "Backgrounder only" is a `tsm` configuration change that starts only
   that process on that machine; this is why Section 2's sizing advice —
   scale Backgrounder capacity independently of Gateway/VizQL — is
   directly actionable: you can add a Backgrounder-only node without
   touching viewer-facing capacity at all.
3. HA and DR use different underlying mechanisms even though both are
   "redundancy": HA relies on running duplicate processes of the same
   role behind a load balancer within one deployment, so a live failover
   is automatic and near-instant (the load balancer stops routing to the
   failed node); DR relies on `tsm maintenance backup` producing a
   point-in-time snapshot of the Repository plus file store, shipped to a
   separate deployment — recovery requires restoring that backup and
   promoting the DR site, which is why DR's recovery point is bounded by
   backup frequency (last night's backup) rather than being continuous
   like HA.
4. Capacity planning inputs are additive but not interchangeable in the
   underlying resource model: concurrent viewers and workbook complexity
   both consume VizQL Server CPU/memory at query time, while extract
   volume/frequency consumes Backgrounder CPU/memory on a schedule — since
   these are different processes (Section 2), a deployment can be
   correctly sized for one and badly undersized for the other
   simultaneously, which is exactly the scenario Section 4's common
   mistake and the Exercise both test.

## Cheat sheet

| Concern | Key architecture decision |
|---|---|
| Tenant isolation | Single site + strong permissions vs. multi-site |
| Process placement | Separate Gateway/VizQL from Backgrounder for large refresh volume |
| Resilience | HA (node redundancy) vs. DR (separate site + backup) |
| Sizing | Viewer count AND Backgrounder/refresh load, not just one |

## Exercise

A company runs 300 nightly extract refreshes, each roughly the complexity
of the Level 3 Module 4 `Orders` flow, all scheduled in a 1-hour window,
alongside 50 concurrent daytime viewers. Which capacity input (Section 4)
is most likely under-provisioned if the nightly refresh window regularly
overruns, and which node role (Section 2) should be scaled first? (The
refresh-volume input is under-provisioned; scale Backgrounder capacity,
not Gateway/VizQL, since the viewer count of 50 is comparatively light.)
