# 03 · Row-Level Security

Row-level security (RLS) restricts *which rows* of `Orders` a given viewer
sees, rather than hiding whole dashboards. This module builds RLS using a
user-mapping table and verifies it with hand arithmetic.

## 1. The entitlement table

`RegionAccess`, mapping Tableau Server usernames to the Region(s) they may
see:

| Username | Region |
|---|---|
| alice@co.com | East |
| bob@co.com | West |
| carol@co.com | Central |
| admin@co.com | East |
| admin@co.com | West |
| admin@co.com | Central |

Admin gets three rows (one per Region) — a common pattern for a manager who
should see everything.

## 2. Approach 1: USERNAME() filter with a joined entitlement table

1. Join `Orders` to `RegionAccess` on `Region`, and add a calculated field
   `Region Is Visible`:

    ```
    [Username] = USERNAME()
    ```

2. Place `Region Is Visible` on the Filters shelf, set to **True**. When
   `alice@co.com` views the dashboard, `USERNAME()` evaluates to
   `"alice@co.com"`, matching only the `RegionAccess` row for East — so
   the join keeps only `Orders` rows where Region = East.
3. Verify: Alice should see exactly the 3 East orders (1001, 1003, 1006)
   summing to 2210 — not the full 6880. Bob sees the 3 West orders (1002,
   1005, 1008) summing to 3750. Carol sees the 2 Central orders (1004,
   1007) summing to 920.
4. Admin has three `RegionAccess` rows, one per Region, so the join
   matches all three Regions for `admin@co.com` — admin sees all 8 orders,
   total 6880.

## 3. Approach 2: ISMEMBEROF() with published groups

1. Alternative when access maps to a Server **group** rather than
   individual usernames: create groups `RLS-East`, `RLS-West`,
   `RLS-Central` on Server, and a calculated field:

    ```
    (ISMEMBEROF('RLS-East') AND [Region] = "East")
    OR (ISMEMBEROF('RLS-West') AND [Region] = "West")
    OR (ISMEMBEROF('RLS-Central') AND [Region] = "Central")
    ```

2. This avoids maintaining a per-user entitlement table inside the data
   source — group membership is managed on Server instead (Module 2's
   Groups), which is typically easier to keep current as staff change.

## 4. Verifying RLS didn't break aggregate math

1. Common bug: forgetting to make the entitlement join filter apply
   correctly for a user who belongs to **no** Region (e.g. a new hire not
   yet added to `RegionAccess`) — such a user should see **zero** rows,
   not all rows. Test by hand-checking a username absent from
   `RegionAccess`: the join produces no matching row, `Region Is Visible`
   evaluates to false for every row, and the filter (set to True) removes
   everything — confirm the dashboard shows a blank/zero state rather than
   silently falling back to unrestricted data.
2. Test each of the three real users again after any calculation change:
   Alice's total must stay 2210, Bob's 3750, Carol's 920 — any drift means
   the RLS filter or its underlying join broke.

## 5. Performance note

1. RLS filters (like context filters) run on every query — a heavily
   nested `ISMEMBEROF`/`USERNAME()` calculation evaluated against a large
   fact table benefits from being marked as a **context filter** so it's
   computed once and cached, rather than re-evaluated per user interaction
   (Level 2 Module 8).

## Cheat sheet

| Approach | Best for |
|---|---|
| `USERNAME()` + joined entitlement table | Per-user, data-driven access |
| `ISMEMBEROF('group')` | Access maps cleanly to Server groups |
| Filter shelf, set to True | Where the RLS calc is applied |
| Context filter | Performance + correct filter ordering |
| Zero-row test for unmapped user | Confirms fail-closed, not fail-open |

## Exercise

A new user `dana@co.com` is added to `RegionAccess` with two rows: Region
"East" and Region "West". Hand-compute what Dana should see: the union of
East (1001, 1003, 1006 = 2210) and West (1002, 1005, 1008 = 3750) orders,
totalling 2210+3750 = 5960 — and explain why Central's 920 should not
appear.
