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

## Cheat sheet

| CoE function | Analogous course module |
|---|---|
| Governance | Level 3 Module 8 — certification |
| Enablement | Training authors in course techniques (LOD, RLS, Prep) |
| Platform | Level 4 Module 1 — architecture/capacity |
| Support | Resolving discrepancies like the 6880 vs. 6700 case |

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
