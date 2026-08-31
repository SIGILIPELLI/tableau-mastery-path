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
