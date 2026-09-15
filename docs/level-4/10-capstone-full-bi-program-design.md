---
description: "Capstone — Full BI Program Design — Final capstone: design a complete BI program for 'Northwind Retail' (the company behind this course's running dataset)…"
---

# 09 · Capstone — Full BI Program Design

Final capstone: design a complete BI program for "Northwind Retail"
(the company behind this course's running dataset) as it scales from a
single analyst's spreadsheet to an enterprise deployment — integrating
every Level 4 module and hand-verifying every number the design assumes.

## 1. Starting point and requirements

1. Northwind Retail currently has: the 8-row `Orders` table (Level 1),
   three regional managers who each need to see only their own Region
   (Level 3 Module 3's `RegionAccess`), and a growth plan to expand from
   3 regions to a national deployment with an estimated 300 concurrent
   viewers and 300 nightly extract refreshes within 2 years.
2. Ground truth to design against: grand total Sales 6880 (East 2210 +
   West 3750 + Central 920); per-category totals Furniture 4050,
   Electronics 2650, Office Supplies 180; per-category profit Furniture
   386, Electronics 420, Office Supplies 60 (Level 3 Module 5, Section 5).

## 2. Architecture decision (Module 1)

1. Given a single company (not multiple isolated tenants) and moderate
   admin capacity, recommend **single-site** with strong project
   permissions (Level 3 Module 2) over multi-site — Northwind doesn't have
   the regulatory/tenant-isolation need that would justify multi-site's
   overhead.
2. Given the projected 300 nightly refreshes (matching Module 1's
   exercise exactly), size Backgrounder capacity explicitly for refresh
   volume, not just the 300 viewer count, and stagger refresh windows
   (Module 4) rather than scheduling all 300 at once.

## 3. Governance design (Modules 2–3, Level 3 Module 8)

1. Certify exactly one `Orders` source (the 6880-total, 8-row source) as
   the single source of truth before allowing any self-service dashboard
   development against it — following Level 3 Module 8's review process:
   any candidate source must re-derive to 6880 grand total and the correct
   Region breakdown before certification.
2. Stand up a lightweight CoE (Module 2) with governance (certification
   owner), enablement (training new analysts on hand-verification habits),
   and platform (Module 1's architecture) functions — for a company this
   size, one or two people may cover all three functions initially, with
   headcount added as the CoE maturity model (Module 2, Section 3)
   advances from Managed to Governed.

## 4. Self-service and RLS design (Module 5, Level 3 Module 3)

1. Bake RLS into the certified source itself (Level 3 Module 3's
   `RegionAccess` join) so every self-service dashboard built on it
   automatically respects Region scoping — a new regional manager for a
   4th region ("North," say) added to `RegionAccess` should immediately
   see only their region's rows in any dashboard built on the certified
   source, with zero per-dashboard RLS work by the dashboard's author.
2. Verify design correctness with the same test as Level 3 Module 3:
   Alice (East) must see exactly 2210; Bob (West) 3750; Carol (Central)
   920; an admin-role user all three regions summing to 6880.

## 5. Cost and license plan (Module 8)

1. Right-size licenses from the start rather than retrofitting later
   (Module 8, Section 2): estimate role mix at scale (a handful of
   Creators building/maintaining the certified sources and core
   dashboards, a larger Explorer tier for self-service authors, Viewer for
   the bulk of the 300 projected concurrent users) and apply the same
   cost-unit reasoning — a Creator-for-everyone default at 300 users would
   be dramatically more expensive than a right-sized mix, exactly as in
   Module 8's 50-person worked example.

## 6. Full program design checklist

| Layer | Decision | Verification |
|---|---|---|
| Architecture | Single-site, Backgrounder sized for 300 nightly refreshes | Module 1's exercise reasoning |
| Data governance | One certified `Orders` source | Re-derives to 6880 / 2210 / 3750 / 920 |
| Organizational | Lightweight CoE, 3 functions | Module 2's maturity model, aiming for Governed |
| Security | RLS baked into certified source | Alice 2210, Bob 3750, Carol 920, admin 6880 |
| Cost | Role-mix license tiering from day one | Module 8's cost-unit comparison |

## How It Actually Works

1. Every design decision in this capstone traces to a specific mechanism
   covered earlier in the course, and the checklist (Section 6) is really
   a map of which stored artifact each decision lives in: the architecture
   choice is a Server topology/site configuration; certification is a
   metadata flag on the datasource content record (Level 3 Module 8);
   RLS is a join plus a `USERNAME()`-driven filter on the certified
   source's own definition (Level 3 Module 3); licensing is a per-user
   role attribute checked at the session-authorization layer (Level 4
   Module 8) — none of these are abstract policies, each is a concrete
   object Tableau stores and enforces mechanically.
2. Because RLS is attached to the certified source's connection (not to
   any individual dashboard), adding North to `RegionAccess` propagates to
   every workbook built against that source automatically on next query —
   this is the same connection-reference mechanism from Level 4 Module 6:
   a self-service author's dashboard references the source's content ID,
   so the source's own filter logic (including the newly added North row)
   applies without the author republishing anything.
3. The capstone's two-update answer (re-certify the source, update
   `RegionAccess`) is forced by query-evaluation order (Level 3 Module 10):
   if only `RegionAccess` is updated but the certified source's underlying
   extract isn't refreshed/re-certified to include orders 1009/1010, the
   new North rows don't exist in the queried table at all — RLS can only
   filter *existing* rows, it can't conjure rows a stale extract never
   pulled. Conversely, if only the extract is refreshed but no North
   manager is added to `RegionAccess`, the North rows exist in the source
   but every RLS-scoped query (`WHERE Username = USERNAME()`) returns zero
   matching rows for anyone querying as a manager without a North mapping
   — updating just one of the two layers leaves the system in a broken
   intermediate state, which is exactly why both updates are required
   together, not sequentially with either one skipped.
4. Backgrounder sizing for 300 nightly refreshes (Section 2) and license
   role-mix (Section 5) both scale from the same underlying capacity
   model established in Level 4 Modules 1 and 8 — refresh cost is driven
   by job count × per-job duration regardless of headcount, while license
   cost is driven by role mix × headcount regardless of refresh volume;
   designing both "from day one" (rather than retrofitting) avoids the
   exact overrun and over-licensing failure modes those modules' worked
   examples diagnose after the fact.

## Exercise (final capstone check)

Northwind adds a 4th region, North, with 2 new orders: Order 1009 (North,
Furniture, Sales 500) and Order 1010 (North, Electronics, Sales 300).
Recompute the full program's ground-truth numbers after this addition:
new grand total = 6880 + 500 + 300 = **7680**; new region breakdown East
2210, West 3750, Central 920, North 800; new Furniture category total =
4050 + 500 = 4550; new Electronics category total = 2650 + 300 = 2950.
State which two layers of the design (Section 6) must be updated for this
change to be handled correctly end-to-end: the certified data source
itself (re-certify with the new rows and updated totals, per Level 3
Module 8) and the RLS entitlement table (`RegionAccess` needs a new North
manager's username mapped to Region = North, per Level 3 Module 3) —
without both updates, the new region's data would either be missing from
the trusted source or visible to no regional manager at all.
