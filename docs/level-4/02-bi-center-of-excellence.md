---
description: "Building a BI Center of Excellence — A BI Center of Excellence (CoE) is the organizational structure that turns scattered Tableau usage (like the ad-hoc…"
---

# 01 · Building a BI Center of Excellence

A **BI Center of Excellence (CoE)** is the organizational structure that
turns scattered Tableau usage (like the ad-hoc `Orders` copies from Level 3
Module 8) into a governed, consistent practice. This module covers what a
CoE does and how to measure whether it's working.

## 1. Why a CoE exists

1. Recall Level 3 Module 8's problem: three analysts published three
   different `Orders` totals (6880 correct, 6700 and 5780 wrong) with no
   process to catch the discrepancy before it reached a viewer. A CoE is
   the standing team/function whose job is to prevent that class of
   problem at scale — not by personally checking every workbook, but by
   setting the certification process (Module 8), training, and support
   structure that make correct-by-default the easy path.
2. A CoE is not the same as a single admin doing Server maintenance
   (Level 3 Module 2) — it typically includes governance (certification
   process owners), enablement (training/support for self-service
   authors), and platform (capacity/architecture from Module 1) functions,
   which can be one overloaded person in a small org or several dedicated
   roles in a large one.

## 2. Core CoE functions

1. **Governance**: owns the certification workflow (Level 3 Module 8),
   naming/tagging standards, and content lifecycle policy (archiving
   Analyst B's and C's incorrect `Orders` copies once the CoE identifies
   them).
2. **Enablement**: trains new Creators on the patterns this course covers
   — e.g. ensuring every new analyst knows to hand-verify a data source's
   grand total (6880) before building on it, per the review habit in
   Level 3 Module 8, Section 5.
3. **Platform**: owns architecture and capacity decisions from Module 1 —
   sizing Backgrounder for refresh volume, deciding single- vs. multi-site.
4. **Support**: the escalation path when a viewer finds a discrepancy —
   e.g. someone reporting "I see 6700, not 6880" should have a clear
   channel to the CoE, which then re-derives the correct total from raw
   rows (as in Level 3 Module 8's exercise) before resolving.

## 3. Maturity model

| Stage | Characteristics |
|---|---|
| Ad hoc | Anyone publishes anything; no certification; duplicate/conflicting sources (Level 3 Module 8's 3-analyst problem) common |
| Managed | Certification process exists but is inconsistently followed; some governed, some shadow sources |
| Governed | Certification enforced for production dashboards; clear ownership; lineage tracked (Level 3 Module 8) |
| Optimized | Governance is largely self-service — Creators default to certified sources; CoE spends most effort on enablement/platform, not firefighting |

## 4. Metrics a CoE should track

1. **% of published data sources certified** — a rising trend indicates
   the governance process is being adopted rather than bypassed.
2. **Duplicate source count** — e.g. counting how many `Orders`-equivalent
   sources exist server-wide; ideally converging toward 1 certified source
   plus legitimate departmental variants, not 3+ near-duplicates with
   silently different totals.
3. **Time-to-certify** — how long a new data source waits between
   submission and certification review; too long encourages authors to
   skip the process and publish uncertified.
4. **Support ticket volume tied to data discrepancies** — a proxy for how
   often the ad-hoc-stage problem (conflicting totals) is still reaching
   end users despite governance.

## 5. Common CoE anti-patterns

1. **Governance without enablement**: mandating certification but not
   training analysts on *how* to validate a source (e.g. skipping the
   hand-verification habit from Level 3 Module 8) — produces slow,
   resented gatekeeping rather than faster, more trustworthy publishing.
2. **CoE as sole bottleneck**: routing every single workbook through a
   small central team to certify, rather than training domain teams to
   self-certify against clear standards — doesn't scale past a handful of
   data sources.

## How It Actually Works

1. The CoE metrics in Section 4 aren't manually tallied — they're pulled
   from the same Metadata API graph that powers lineage (Level 3 Module 8):
   "% certified" is `COUNT(datasources WHERE certificationNote IS NOT
   NULL) / COUNT(all published datasources)`, run as a GraphQL query
   against the catalog, and "duplicate source count" is a fuzzy match over
   published data source names/schemas (e.g. clustering the 12 "Orders"/
   "Sales Data" sources in the exercise by shared column names like
   `Region`, `Sales`, `Profit`) — the CoE tooling surfaces candidates, but
   the actual dedup call (which of the 12 is correct) still requires the
   row-level re-derivation habit from Level 3 Module 8, Section 5, since
   two sources can share a schema and still disagree on totals.
2. Time-to-certify is measured off two timestamps on the content item's
   REST API record: the datasource's `createdAt`/publish event and the
   `certification` field's set time (captured in Server's audit/usage
   tables, `historical_events` in the Postgres repository) — subtracted
   per source and aggregated (median, not mean, since a few multi-week
   stragglers otherwise dominate the average and mask a healthy typical
   turnaround).
3. Maturity-model movement (Section 3) is observable, not just descriptive:
   an "Ad hoc" org's Metadata API graph shows many datasource nodes with
   near-identical schemas and no certification edges; a "Governed" org's
   graph shows most workbook nodes' data-source edges terminating at a
   small number of certified nodes — so a CoE can literally query "what
   fraction of workbook→datasource edges point at an uncertified node" as
   a maturity proxy, the graph-level version of the duplicate-count metric
   in Section 4.
4. The support-ticket metric closes the loop mechanically: a reported
   discrepancy ("I see 6700, not 6880") is resolved the same way Level 3
   Module 8's exercise resolves it — pull both sources' raw rows, re-sum by
   Region, and locate exactly which row(s) differ — then, if the
   discrepancy traces to a specific known-uncertified duplicate, the fix
   is recorded against that node in the catalog (deprecation warning or
   archival) so the *next* viewer who searches "Orders" doesn't rediscover
   the same 6700 total independently.

## Cheat sheet

| CoE function | Analogous course module |
|---|---|
| Governance | Level 3 Module 8 — certification |
| Enablement | Training authors in course techniques (LOD, RLS, Prep) |
| Platform | Level 4 Module 1 — architecture/capacity |
| Support | Resolving discrepancies like the 6880 vs. 6700 case |

## 🔀 Related lessons on other tracks

- [Agile — 05 · Building an Internal Agile Center of Excellence](https://sigilipelli.github.io/agile-mastery-path/level-4/05-internal-agile-center-of-excellence/)
- [AI Tools — 07 · Building an AI Center of Excellence](https://sigilipelli.github.io/ai-tools-mastery-path/level-4/07-building-ai-center-of-excellence/)
- [Claude Training — 08 · Building an AI Usage Center of Excellence](https://sigilipelli.github.io/claude-training-mastery-path/level-4/08-ai-center-of-excellence/)

## Exercise

A CoE inherits a Server with 12 data sources named some variant of
"Orders" or "Sales Data," none certified. Outline the first three steps
you'd take (in order) before certifying any of them, using Level 3 Module
8's verification approach as a model. (1. Pull each source's grand total
and per-region breakdown, re-deriving from raw rows rather than trusting
the displayed total. 2. Identify which — if any — matches the known-correct
6880/2210/3750/920 structure. 3. Certify the one correct source, flag or
archive the rest, and document why, before opening any certification
workflow to future authors.)
