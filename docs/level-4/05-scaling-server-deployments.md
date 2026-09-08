# 04 · Scaling Tableau Server Deployments

Building on Module 1's architecture, this module covers the operational
side of scaling an existing deployment as usage grows — adding nodes,
load-testing, and monitoring — using the same `Orders`-scale workload as a
concrete (if tiny) reference point for the reasoning.

## 1. Vertical vs. horizontal scaling

1. **Vertical scaling**: adding CPU/RAM to an existing node — the simplest
   first step when a single VizQL Server node is CPU-bound rendering
   dashboards like the Level 3 Module 10 RLS project for many concurrent
   viewers.
2. **Horizontal scaling**: adding more nodes running the same process
   (e.g. a second and third VizQL Server node) behind a load balancer —
   needed once vertical scaling hits diminishing returns or a single
   machine's ceiling, and the approach that also improves availability
   (Module 1, Section 3).

## 2. Identifying the bottleneck before scaling

1. Don't scale blindly — use the Performance Recording approach (Level 3
   Module 5) at the server level: Tableau Server's **Admin Views**
   (TS status pages) break down load by process (VizQL, Backgrounder,
   Data Server) so you can see which one is actually saturated.
2. Worked reasoning: if Backgrounder is at 95% utilization overnight
   running extract refreshes (e.g. many `Orders`-style incremental
   refreshes per Level 3 Module 4) while VizQL sits at 20% during the day,
   adding VizQL nodes (Section 1) does nothing for the actual bottleneck —
   the fix is more Backgrounder capacity or refresh schedule spreading,
   per Module 1 Section 2's sizing guidance.

## 3. Load testing before a scale-up commitment

1. Tableau's **TabJolt** (or equivalent load-testing approach) replays a
   representative mix of viewer interactions against a staging deployment
   to measure response time under increasing concurrent load, before
   committing to a production topology change.
2. A meaningful test workload should reflect real usage patterns — testing
   only simple views (a single Region bar chart) when production
   dashboards are RLS-gated, LOD-heavy pages like Level 3 Module 10's
   project understates real load, since RLS filters (Level 3 Module 3,
   marked as context) and nested FIXED LODs (Level 3 Module 1) cost more
   per render than a plain bar chart.

## 4. Extract refresh scheduling at scale

1. As the number of published extracts grows, naive "everything refreshes
   at 2 AM" scheduling creates a Backgrounder traffic spike — spreading
   refresh windows (e.g. staggering extracts across 2 AM–6 AM based on
   dependency order and priority) avoids the overrun scenario from Module
   1's exercise (300 refreshes crammed into 1 hour).
2. Incremental refresh (Level 3 Module 4) reduces per-run cost but doesn't
   eliminate the need for scheduling discipline — 300 incremental refreshes
   still each carry some fixed overhead (connection setup, metadata
   checks) even if the row-level work is small.

## 5. Monitoring and alerting

1. Key metrics to alert on: Backgrounder queue depth/backlog (a rising
   trend indicates refresh capacity is falling behind schedule), VizQL
   response time percentiles (p95/p99, not just average — a slow-render
   worksheet like a table-calc-heavy view, Level 3 Module 5, can be masked
   by a fast average if most other views are simple), and Repository
   (Postgres) health, since it's a single point of failure for metadata
   even in an otherwise horizontally-scaled deployment.

## How It Actually Works

1. Admin Views are built from Tableau Server's own instrumented
   `historical_events`/`background_jobs` tables in the Repository
   (Postgres) — every VizQL render, extract refresh, and metadata
   operation logs a row with process, node, start/end time, and status,
   which is what the Admin Views dashboards query and aggregate. This is
   why "look at Admin Views before scaling" is not just good practice but
   the only reliable way to identify the bottleneck: the alternative
   (guessing which process is saturated from symptoms alone) is exactly
   the mistake Module 1's exercise warns against — sizing for viewer count
   while a Backgrounder queue silently backs up.
2. Horizontal scaling works because VizQL Server and Backgrounder are both
   stateless per-request processes coordinated through the shared
   Repository and a load balancer/dispatcher — adding a second VizQL node
   means the load balancer round-robins (or least-connections) incoming
   view requests across both nodes, each capable of independently
   compiling and running a query, so throughput scales roughly linearly
   until the shared Repository or the data source itself becomes the new
   bottleneck. Backgrounder scales the same way, except jobs are pulled
   from a shared queue rather than load-balanced per request.
3. p95/p99 latency, not average, is the right monitoring signal because
   Admin Views' render-time data is heavily right-skewed: many simple
   worksheets render in well under a second, while a small number of
   LOD/table-calc-heavy dashboards (Level 3 Module 5) dominate the tail.
   Averaging masks this — 99 renders at 0.3s and 1 render at 8s averages
   to ~0.38s, hiding the exact outlier that's generating complaints, which
   is the concrete mechanism behind the Exercise: 30% average utilization
   with an 8s p99 on one dashboard is a workbook-level calc cost problem,
   not aggregate capacity, because *most* requests aren't touching that
   dashboard's expensive calc chain at all.
4. Refresh staggering (Section 4) works at the Backgrounder queue level:
   jobs are pulled from the queue up to the configured number of
   concurrent background processes, so if 300 refreshes are all scheduled
   for the same instant, they queue and process serially/in limited
   parallel regardless — spreading start times across a window doesn't
   reduce total work, it reduces the *peak* queue depth at any one moment,
   which is what actually prevents the overrun scenario, since a queue
   that never exceeds available Backgrounder capacity drains continuously
   instead of falling further behind each cycle.

## Cheat sheet

| Symptom | Likely bottleneck | Scaling response |
|---|---|---|
| Slow dashboard render, viewers complain during business hours | VizQL Server saturated | Add VizQL nodes (horizontal) or vertical scale |
| Refresh jobs run late/queue backs up overnight | Backgrounder saturated | Add Backgrounder capacity, stagger schedules |
| Metadata operations (publish, permissions) slow | Repository (Postgres) under load | Vertical scale Repository node, check for backup/vacuum conflicts |
| All processes fine individually, but content-level views slow | Calc complexity (LOD/table calc heavy) | Optimize workbook (Level 3 Module 5), not infrastructure |

## Exercise

Admin Views show VizQL Server at 30% average utilization but p99 response
time is 8 seconds on a specific dashboard, while every other dashboard
responds in under 1 second. Using Section 5's guidance, explain why adding
VizQL nodes is unlikely to fix this specific complaint, and what you'd
check instead. (Low average utilization with one slow outlier points to a
workbook-level problem — e.g. an unoptimized table calc or FIXED LOD
chain (Level 3 Module 5) — not insufficient server capacity; check that
dashboard's Performance Recording before scaling infrastructure that
isn't actually the bottleneck.)
