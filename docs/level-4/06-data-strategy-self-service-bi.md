---
description: "Data Strategy & Self-Service BI Enablement — Self-service BI means giving analysts across an organization the ability to build their own dashboards on…"
---

# 05 · Data Strategy & Self-Service BI Enablement

Self-service BI means giving analysts across an organization the ability
to build their own dashboards on governed data, without every request
routing through a central team. This module covers how to make that safe,
using the certified `Orders` source and CoE structure from Modules 2–3.

## 1. The self-service spectrum

1. **Fully centralized**: only a BI team builds anything; every request is
   a ticket. Safe but slow, and doesn't scale past a small number of
   dashboards.
2. **Fully decentralized (uncontrolled)**: anyone connects to raw source
   tables and builds whatever they want — the Level 3 Module 8 scenario
   (three conflicting `Orders` totals) is the predictable outcome at
   scale.
3. **Governed self-service**: analysts build their own workbooks, but
   against certified data sources (Level 3 Module 8) with governed
   calculated fields (Level 4 Module 3) — the goal state most mature CoEs
   aim for.

## 2. What makes self-service safe: the certified data source as the gate

1. If every self-service author connects to the certified `Orders` source
   (grand total 6880, verified in Level 3 Module 8) rather than raw
   tables, their individual dashboards inherit correctness by
   construction — an author building a new "Profit by Category" view atop
   the certified source should get Furniture 386, Electronics 420, Office
   Supplies 60 (Level 3 Module 5, Section 5) without re-deriving those
   numbers themselves, because the certified source's row-level Profit
   values are already correct.
2. This is the practical payoff of Modules 2–3's governance investment:
   self-service scales precisely because the guardrail (certification) is
   upstream of individual authors, not something each author has to
   re-establish per workbook.

## 3. Enablement: training self-service authors

1. Minimum skill baseline for a self-service author, mapped to this
   course: distinguishing dimensions/measures and live/extract (Level 1),
   writing basic calculated fields and using filters correctly including
   context filters (Level 2), and knowing when a data source is certified
   vs. not (Level 3 Module 8) before building on it.
2. A common enablement gap: authors who can build visually compelling
   dashboards but don't know to hand-verify a number before publishing
   (the habit this entire course has built) — training should explicitly
   include "how do I know this number is right," not just tool mechanics.

## 4. Self-service guardrails without recreating the bottleneck

1. **Row-level security by default** on sensitive certified sources
   (Level 3 Module 3) — a self-service author building on the certified
   `Orders` source shouldn't need to build RLS themselves; it should
   already be enforced at the source, so every workbook built on it
   inherits the same Region-scoping automatically.
2. **Publish-time validation**, not build-time restriction — e.g. a
   lightweight automated check that a newly published workbook's key
   metric (grand total Sales) falls within an expected range compared to
   the certified source's own total, flagging (not blocking) anomalies
   for CoE review, similar in spirit to Level 3 Module 8's manual
   discrepancy check but automated.

## 5. Measuring self-service health

| Metric | What it indicates |
|---|---|
| % of workbooks built on certified sources | Whether the gate in Section 2 is actually being used |
| Ratio of self-service to BI-team-built dashboards | How much load has shifted off the central team |
| Discrepancy reports per month (Level 4 Module 2 metric) | Whether governance is holding up as self-service scales |
| Time from data need to published dashboard | The actual speed benefit self-service is meant to deliver |

## How It Actually Works

1. "Inheriting correctness by construction" (Section 2) is a real
   connection-metadata mechanism, not just a policy statement: when an
   author connects to the certified `Orders` source rather than a raw
   table, Tableau records a reference to that published data source's
   content ID inside the new workbook's `.twb`, the same reference
   mechanism used in Level 3 Module 10's capstone. Every calculation the
   author writes (e.g. their own `Profit by Category` view) executes
   against whatever rows and row-level security the certified source
   currently defines — RLS included, since it's attached to the source's
   connection, not to each individual workbook — so an author never has
   to re-implement `RegionAccess` filtering themselves; it rides along
   with the connection.
2. Publish-time validation (Section 4.2) is implementable as a scheduled
   job using the same REST/Metadata API surfaces the CoE metrics use
   (Level 4 Module 2): query each newly published workbook's underlying
   view data (via the REST API's "query view data" endpoint, or a
   scripted comparison against the certified source's own total),
   re-derive its grand total, and flag any workbook whose total falls
   outside an expected tolerance of the certified 6880 — this is
   mechanically the same hand-verification habit from Level 3 Module 8,
   just automated as a diff check rather than performed by a person per
   workbook.
3. The Exercise's 12-dashboard risk is concrete because a raw-table
   connection has no certification metadata attached at all — Tableau has
   no way to distinguish "this raw copy happens to currently match 6880"
   from "this raw copy is stale/filtered" at connection time; the
   discrepancy only becomes visible when someone (or an automated
   publish-time check) actually re-sums its rows, exactly as in Level 3
   Module 8's original three-analyst scenario, just at 12x the blast
   radius since each of the 12 workbooks independently re-queries the
   same unvetted raw table rather than sharing one governed reference.
4. Migration remediation (Level 4 Module 3's deprecation workflow) is the
   same connection-metadata swap as any republish: repointing each
   workbook's data source reference from the raw table to the certified
   source's content ID, then re-running the same total/region check
   (6880, 2210/3750/920) as a regression test — if any of the 12 fails to
   match post-migration, the mismatch identifies exactly which workbook's
   old raw-table logic (a stray filter, a different join) diverged from
   the certified definition, which is information the migration step
   itself surfaces almost for free.

## Exercise

An organization has 40 self-service-built dashboards; an audit finds 12
connect directly to a raw, uncertified copy of `Orders` rather than the
certified source. Using Section 2's reasoning, explain the specific risk
this creates (each of those 12 dashboards could silently diverge from the
certified 6880 total if the raw copy is stale or filtered differently,
recreating Level 3 Module 8's multi-source problem at a larger scale) and
the remediation step that follows Level 4 Module 3's deprecation workflow
(migrate the 12 dashboards to the certified source, verify each still
reports 6880/2210/3750/920, then restrict or deprecate direct access to
the raw table).
