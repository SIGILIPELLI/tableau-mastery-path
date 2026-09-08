# 02 · Advanced Governance & Content Strategy

Building on Level 3 Module 8's certification and Level 4 Module 2's CoE,
this module covers content strategy at scale: lifecycle policy, tagging
taxonomy, and deprecation, using the `Orders`-family sources as the
running example.

## 1. Content lifecycle stages

1. **Draft** — an analyst is building against `Orders` (or a certified
   copy of it), not yet published for others.
2. **Published, uncertified** — shared to Server but not yet reviewed;
   this is where Level 3 Module 8's three conflicting analyst copies
   (6880, 6700, 5780) would sit before governance review.
3. **Certified** — reviewed and marked as the trusted source, per Level 3
   Module 8.
4. **Deprecated** — still accessible but flagged for retirement (e.g. the
   old `OrderFacts`/`RegionDim` split from Level 3 Module 6 being replaced
   by a newer certified single-table source).
5. **Archived/removed** — no longer accessible; only performed after
   impact analysis (Level 3 Module 8, Section 4) confirms nothing
   published still depends on it.

## 2. Tagging and naming taxonomy

1. A consistent taxonomy makes discovery scale — e.g. tagging every
   Northwind-related source with `domain:sales`, `region:all` or
   `region:east` for RLS-scoped variants, and `status:certified`.
2. Naming convention example: `[Domain] Subject — Grain` — e.g. "Sales
   Orders — Order Line" for the row-level `Orders` table, vs. "Sales
   Orders — Region Summary" for a pre-aggregated Region-level extract (the
   3-row East/West/Central summary used in several Level 3 modules) — the
   grain suffix prevents someone joining or comparing two sources at
   different grains and getting confused by why a summary source's 3 rows
   don't match a detail source's 8.

## 3. Deprecation workflow

1. Before deprecating the old `OrderFacts`/`RegionDim` two-table source in
   favor of a newer single joined certified source, run impact analysis
   (Level 3 Module 8, Section 4) to find every workbook still connected to
   the old tables.
2. Notify owners of dependent workbooks with a concrete migration path and
   a deadline; verify post-migration that totals match — e.g. a dashboard
   migrated from the old split source to the new one should still report
   East 2210, West 3750, Central 920, grand total 6880; any drift signals
   a join or filter difference introduced during migration, not an
   acceptable "close enough."
3. Only after all dependents have migrated (confirmed via lineage,
   ideally re-run at deprecation-plus-30-days) does the old source move to
   Archived/removed.

## 4. Governance for calculated fields and metrics

1. Reusable calculated fields (e.g. `Category Sales in Region` from Level
   3 Module 1) proliferate copy-pasted and subtly modified across
   workbooks if not governed — a **certified/shared calculation** (via a
   published data source's defined fields, or Tableau's metrics
   definitions) keeps one authoritative formula rather than N slightly
   different reimplementations.
2. Concrete failure mode without this governance: one team's copy of
   `Pct of Region` divides by `{FIXED [Region]: SUM([Sales])}` (correct,
   matches Level 3 Module 1's 97.3%/70.7%/87.0%), while another team's
   copy accidentally divides by the grand total 6880 instead — producing
   East 2210/6880 = 32.1%, a different and less useful number that looks
   plausible enough to go unnoticed without governance catching the
   formula drift.

## 5. Content strategy scorecard

| Dimension | Poor | Good |
|---|---|---|
| Lifecycle clarity | No visible stage; everything looks equally "official" | Draft/Published/Certified/Deprecated visibly tagged |
| Naming | "Orders2," "Orders_final," "Orders_final_v3" | Consistent domain–subject–grain convention |
| Calculation reuse | Formula copy-pasted and drifted across workbooks | Shared/certified calculated fields, single source of truth |
| Deprecation discipline | Old sources silently left online indefinitely | Time-boxed migration + impact analysis before archive |

## How It Actually Works

1. Lifecycle stage isn't a free-text label — it's implemented as a
   combination of the `certification` metadata field (Level 3 Module 8)
   plus a project/tag convention (e.g. a `Deprecated` project or tag that
   governance tooling and search filters both key off). Moving a source
   through Draft → Published → Certified → Deprecated → Archived is
   literally a sequence of REST API / UI calls updating those same
   metadata fields — there's no separate "lifecycle engine"; the stages
   are a policy layered on top of certification and tagging primitives
   already covered in Level 3 Module 8.
2. Tags and naming are matched by Server's search index at query time —
   `domain:sales` is a first-class tag object attached to the content
   item, indexed so a search or Metadata API filter can retrieve "every
   datasource tagged domain:sales" in one query. Grain-suffix naming
   ("— Order Line" vs. "— Region Summary") is *not* machine-enforced by
   Tableau itself; it's a convention that only prevents confusion because
   humans read it before joining — which is why Section 2's example
   pairs a naming convention with an actual grain check: a summary
   source's 3 rows and a detail source's 8 rows are mechanically
   different result sets even if both happen to answer "what's the Sales
   total," and no tag stops someone joining them incorrectly.
3. Deprecation's correctness check (Section 3.2) works because migrating
   from the old `OrderFacts`/`RegionDim` split to a new single joined
   source changes *how* the query is generated (Level 3 Module 6's
   join-pushdown mechanics) but must not change the query's *logical*
   result — re-deriving 2210/3750/920/6880 post-migration is a regression
   test against the join semantics themselves, catching the concrete
   failure mode where a migration accidentally changes an inner join to a
   left join (or vice versa) and silently drops or double-counts rows.
4. A "certified/shared calculation" is stored once, on the published data
   source's own definition (or a Tableau Metrics/defined-metric object),
   and every workbook connecting to that source inherits the same
   formula reference rather than a locally re-typed one — this is the
   structural fix for Section 4's drift failure: `Pct of Region` defined
   centrally as `SUM([Sales]) / {FIXED [Region]: SUM([Sales])}` cannot
   independently drift into `SUM([Sales])/6880` in a second workbook,
   because that workbook references the same calculation object rather
   than owning its own copy of the formula text.

## Exercise

Two workbooks report different totals for "East Region Share of Sales":
one shows 97.3%, another shows 32.1%. Using Section 4.2, identify which
formula each workbook is likely using, and state which one matches Level
3 Module 1's definition of `Pct of Region`. (97.3% = 2210/2210-region-total
share of Furniture within East, or more directly East's Sales as a share
of East's own regional total — matches the FIXED-per-Region denominator
definition; 32.1% = 2210/6880, dividing by the grand total instead of the
Region's own total — a drifted, non-matching formula that should be
corrected to the certified definition.)
