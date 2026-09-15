---
description: "Governance & Data Source Certification — Once multiple teams publish workbooks against Orders-like data, governance answers: which data source is the…"
---

# 07 · Governance & Data Source Certification

Once multiple teams publish workbooks against `Orders`-like data, governance
answers: which data source is the trustworthy one, who can change it, and
how do consumers know. This module works through certification and
governance using the published data source from Level 2 Module 9/10.

## 1. The problem certification solves

1. Suppose three analysts each publish their own extract of `Orders` to
   Tableau Server: Analyst A's copy has the full 8 rows (6880 total sales),
   Analyst B's is missing the two Office Supplies rows (accidentally
   filtered, totaling 6880 − 60 − 120 = 6700), and Analyst C's has a stale
   extract from before order 1008 was added (6880 − 1100 = 5780).
2. Without governance, a viewer searching Server for "Orders" finds three
   data sources with three different Sales totals and no way to tell which
   is authoritative — a governance failure that produces silently wrong
   downstream reports.

## 2. Certifying a data source

1. A site admin or designated **certifier** marks one published data
   source (Analyst A's complete, correct 6880-total version) as
   **Certified** — it then shows a badge in the Server/Cloud catalog, and
   Tableau's connection dialog recommends it above uncertified sources
   with a similar name.
2. Certification doesn't technically prevent someone from still using
   Analyst B's or C's uncertified copies — it's a discoverability and
   trust signal, not an access control. Pair certification with actually
   removing or archiving the incorrect duplicates once discovered, since a
   badge alone doesn't fix the 6700 and 5780 totals still sitting on
   Server.

## 3. Data quality warnings

1. Independent of certification, any data source can carry a **data
   quality warning** — e.g. "Warning: this extract's last successful
   refresh was 2024-01-10; the source table has since added rows through
   1008" — shown to every viewer who opens a workbook built on it.
2. Verify the warning's value: if Analyst C's stale source (5780 total,
   missing order 1008's 1100 in Sales) is flagged with a warning, a viewer
   who sees "6880" on a certified dashboard and "5780" with a warning
   banner on Analyst C's has enough information to know which number to
   trust and why they differ (5780 + 1100 = 6880 — exactly the missing
   order).

## 4. Managed metadata: lineage and impact analysis

1. Tableau's **Data Catalog / lineage** view traces a field like `Sales`
   backward to its source column and forward to every workbook/view that
   uses it — for the certified `Orders` source, lineage would show every
   downstream dashboard built on it (e.g. the Level 2 Module 9/10 sales
   dashboard, Level 3 Module 3's RLS dashboard).
2. **Impact analysis** answers "if I change or deprecate this data
   source, what breaks?" — before archiving Analyst B's incorrect 6700-total
   source, lineage confirms whether any published dashboard currently
   depends on it, so removing it doesn't silently break a report someone
   still relies on.

## 5. Governance roles and workflow

1. Typical governance workflow: an analyst builds and validates a data
   source (hand-checking totals as done throughout this course); a
   designated certifier (often a BI team lead) reviews it against known-
   correct totals (6880 grand total, 2210/3750/920 by Region); once
   confirmed, the certifier applies the Certified badge; any subsequent
   change to the certified source ideally goes through the same review
   before republishing, so the badge continues to mean something.
2. This module's dataset makes the review step concrete: a reviewer
   checking a to-be-certified `Orders` source can independently re-derive
   6880 (East 2210 + West 3750 + Central 920) from the raw 8 rows before
   approving — exactly the kind of hand-verification this course has used
   throughout, applied as an actual governance gate.

## How It Actually Works

1. Certification is stored as metadata on the published data source's
   content record on Server/Cloud — technically a flag plus an optional
   certification note, set via the content management page or the REST
   API's `certification`/`certificationNote` fields on a datasource
   resource. Setting it does not touch the `.hyper` file or trigger any
   recompute — it only changes what the Server's search index and catalog
   surface to viewers (badge, ranked higher in "recommended" results), so
   two data sources with the same name but different Sales totals (6880 vs
   6700) remain equally queryable by connection string; certification
   changes discovery, not access.
2. A data quality warning is a separate metadata object attached to the
   content item, with a type (warning, deprecated, stale data, etc.) and
   free-text message; Tableau resolves and displays it at *view render
   time* by checking every data source feeding the workbook being opened,
   which is why the warning banner appears on any dashboard downstream of
   a flagged source — including ones the flagging admin never directly
   touched.
3. Lineage and impact analysis are built from Tableau's **Metadata API**
   (a GraphQL endpoint over the Data Catalog): every published workbook,
   data source, field, and database table is a node, and every "this field
   is used in this worksheet" or "this data source feeds this workbook"
   relationship is a stored edge. A lineage query for the `Sales` field
   walks these edges outward (upstream to the source column, downstream to
   every consuming sheet); impact analysis before deprecating Analyst B's
   6700-total source is the same graph traversal run in reverse — "which
   workbook nodes have an edge into this data source node" — computed from
   the graph, not by re-scanning every workbook's XML on demand.
4. Certification, warnings, and lineage all read from metadata the
   certifier or Server admin must explicitly set or that Tableau infers
   from publish-time parsing — none of them re-validate the *data itself*
   against ground truth. That's exactly why Section 5's workflow puts a
   human hand-recomputation step (independently re-deriving 6880 from the
   8 raw rows) before applying the badge: the certification mechanism has
   no way to know 6700 is wrong on its own, it can only advertise which
   source a human has vouched for.

## Cheat sheet

| Concept | What it does | What it does NOT do |
|---|---|---|
| Certification | Marks the recommended, trusted source | Doesn't block use of other sources |
| Data quality warning | Flags known issues (staleness, etc.) | Doesn't fix the underlying data |
| Lineage | Shows source → field → downstream use | Doesn't prevent breaking changes alone |
| Impact analysis | Shows what breaks before a change | Doesn't make the change safe automatically |

## 🔀 Related lessons on other tracks

- [AI Manager — 04 · Data Governance Basics](https://sigilipelli.github.io/ai-manager-mastery-path/level-2/04-data-governance-basics/)
- [AI Tools — 07 · Data Governance for AI Tool Usage](https://sigilipelli.github.io/ai-tools-mastery-path/level-3/07-data-governance/)
- [Data Engineering — 05 · Data Governance & Cataloging](https://sigilipelli.github.io/data-engineering-mastery-path/level-3/05-data-governance-cataloging/)

## Exercise

A fourth analyst's data source shows a Sales grand total of 6940 — 60
more than the certified 6880. Using the certified source's per-region
breakdown (East 2210, West 3750, Central 920) as ground truth, describe
the governance steps you'd take before either certifying or rejecting the
new source (re-derive its per-region totals from its own row-level data,
locate which row(s) account for the extra 60, and determine whether it
reflects a legitimately new order or a data error) — do not assume either
number is correct without recomputing from raw rows.
