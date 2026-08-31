# 07 · Cost & License Management at Scale

Tableau licensing (Creator/Explorer/Viewer roles, Level 3 Module 2) has
real cost implications at enterprise scale. This module covers license
optimization and cost governance, with worked arithmetic on a small
representative user base.

## 1. License tiers recap and cost shape

1. **Creator**: can build and publish new workbooks/data sources —
   typically the most expensive tier per seat.
2. **Explorer**: can build from published data sources (can't create new
   connections) — mid-cost.
3. **Viewer**: can only view/interact with published content — lowest
   cost per seat, but usually licensed in larger blocks or bundled
   pricing.
4. Cost governance's core question: is each person licensed at the
   *lowest* tier that covers their actual usage, not the tier that's
   simply convenient to assign everyone by default?

## 2. Worked example: right-sizing a 50-person org

1. Suppose a 50-person org has: 5 people who build new data sources and
   workbooks (need Creator), 15 who build dashboards from the certified
   `Orders` source but never create new connections (need Explorer), and
   30 who only view published dashboards like the Level 3 Module 10
   project (need Viewer).
2. If every one of the 50 is licensed as Creator "to be safe," and Creator
   costs (hypothetically, for this arithmetic) 3x an Explorer seat and 6x
   a Viewer seat, the org pays for 50 Creator-equivalents = 300 Viewer-
   equivalent cost units, versus a right-sized mix of 5 Creator (5×6=30
   units) + 15 Explorer (15×3=45 units) + 30 Viewer (30×1=30 units) = 105
   units — a drop from 300 to 105, roughly a 65% reduction, just from
   tier-matching, with no loss of actual capability for anyone.
3. This is the single highest-leverage cost lever in most Tableau
   deployments: auditing actual usage (who has ever published a new data
   source vs. who has only ever built from existing ones vs. who has only
   ever viewed) against assigned license tier.

## 3. Identifying over-licensed users

1. Server/Cloud admin views (Level 4 Module 4's Admin Views) show each
   user's actual activity — a Creator-licensed user with zero "published a
   new data source" events in the last 90 days is a strong candidate for
   downgrade to Explorer, pending confirmation they don't need it
   occasionally (e.g. quarterly, not never).
2. Caution: don't downgrade purely on a snapshot — a Creator who published
   once, 100 days ago, for a legitimate quarterly report, isn't actually
   over-licensed; combine usage history with a conversation, not just a
   dashboard number.

## 4. Extract and storage costs

1. Beyond seat licenses, extract-heavy deployments (Level 4 Module 1's
   Backgrounder sizing) carry infrastructure/storage costs that scale with
   the number and size of `.hyper` extracts — redundant extracts (e.g.
   the Level 3 Module 8 scenario's 3 near-duplicate `Orders` copies, each
   independently refreshed nightly) triple both refresh compute and
   storage cost for data that should exist once, certified.
2. Cost governance therefore overlaps directly with content governance
   (Level 4 Module 3): consolidating duplicate sources isn't just a
   correctness fix, it's a cost optimization, since fewer, well-governed
   extracts mean less redundant Backgrounder and storage spend.

## 5. Building a license/cost review cadence

1. Recommended cadence: quarterly review of (a) license tier vs. actual
   usage per Section 3, (b) duplicate/redundant data source count per
   Level 4 Module 2's metrics, and (c) extract refresh volume trend
   (growing faster than user count growth is a signal worth
   investigating, per Module 4's capacity discussion).

## Cheat sheet

| Lever | Mechanism | Where covered |
|---|---|---|
| Tier right-sizing | Match license to actual usage, not convenience | Section 2 |
| Usage-based downgrade | Admin Views activity history | Section 3 |
| Extract consolidation | Fewer duplicate sources = less refresh/storage cost | Section 4, Level 4 Module 2/3 |
| Quarterly review | Recurring cadence catches drift early | Section 5 |

## Exercise

Using Section 2's cost-unit model, a 20-person team has 8 Creators, 2
Explorers, 10 Viewers. An audit shows 5 of the 8 Creators have never
published a new data source in the last 6 months and only build from
existing certified sources. Recompute the cost-unit total before and
after downgrading those 5 to Explorer, using the ratio Creator=3,
Explorer=1, Viewer=0.5 units: Before = 8(3)+2(1)+10(0.5) = 24+2+5 = 31.
After downgrading 5 Creators to Explorer = 3(3)+7(1)+10(0.5) = 9+7+5 =
**21**, roughly a 32% reduction.
