# 09 · Project — Enterprise Dashboard with Row-Level Security

Capstone for Level 3: build one dashboard combining advanced LOD
calculations (Module 1), a certified/governed data source (Module 8), and
row-level security (Module 3), verified end-to-end with hand arithmetic
against `Orders`.

## 1. Requirements

1. A single dashboard, "Regional Performance," built on the certified
   `Orders` data source (Module 8), showing: (a) SUM(Sales) by Region, (b)
   each Region's share of the Category it sells most, using the nested LOD
   pattern from Module 1, and (c) row-level security so each regional
   manager sees only their own Region's rows, per Module 3's
   `RegionAccess` table.
2. Ground truth to build against (all previously verified in this course):

    | Region | Sales | Top Category (by Sales) | Top Category Sales | Share |
    |---|---|---|---|---|
    | East | 2210 | Furniture | 2150 | 2150/2210 = 97.3% |
    | West | 3750 | Electronics | 2650 | 2650/3750 = 70.7% |
    | Central | 920 | Furniture | 800 | 800/920 = 87.0% |

## 2. Step 1: confirm the data source is certified and correct

1. Open the certified `Orders` source from Module 8 (not an ad-hoc local
   copy) and re-derive the grand total before building anything: 2210 +
   3750 + 920 = **6880**. If this doesn't match, stop and fix the data
   source before building the dashboard — an uncertified/incorrect base
   invalidates every downstream number.

## 3. Step 2: build the Category-share calculation

1. `Category Sales in Region` = `{FIXED [Region],[Category]:
   SUM([Sales])}` (Module 1, Section 2.2).
2. `Region Total` = `{FIXED [Region]: SUM([Sales])}`.
3. `Pct of Region` = `SUM([Sales]) / {FIXED [Region]: SUM([Sales])}`,
   computed at the Region+Category level — reproduces the table in
   Section 1.2: East/Furniture 97.3%, West/Electronics 70.7%,
   Central/Furniture 87.0%.
4. To surface only the **top** category per Region on the dashboard (not
   every category), add a table calc rank (`RANK(SUM([Sales]))` computed
   per Region partition) and filter to Rank = 1 — verify the filtered
   result keeps exactly 3 rows (one per Region), matching Section 1.2's
   table.

## 4. Step 3: apply row-level security

1. Reuse Module 3's `RegionAccess` join and `Region Is Visible` calc
   (`[Username] = USERNAME()`), placed on the Filters shelf set to True,
   applied to every worksheet feeding the dashboard.
2. Re-run Module 3's verification on this specific dashboard: `alice@co.com`
   should see only the East row (2210, Furniture 97.3%); `bob@co.com` only
   West (3750, Electronics 70.7%); `carol@co.com` only Central (920,
   Furniture 87.0%); `admin@co.com` sees all three rows, summing back to
   6880.

## 5. Step 4: performance check before publishing

1. Apply Module 5's diagnostic: since this dashboard mixes FIXED LODs with
   an RLS filter, confirm the RLS filter is marked as a **context filter**
   (Module 3, Section 5) so it's evaluated once and doesn't force the
   FIXED LODs to recompute per interaction incorrectly — and confirm the
   FIXED LODs still return the Section 1.2 numbers *after* marking the RLS
   filter as context (context filters apply before FIXED LODs, per Module
   1 Section 4.3, so Alice's view should now correctly show FIXED
   calculations scoped to her visible East rows only, not the global
   totals leaking through).
2. Explicit check: with RLS as a context filter, Alice's `Region Total`
   FIXED calc should now show 2210 (her own visible total), not 6880 (the
   ungated global total) — if it still shows 6880, the RLS filter isn't
   actually applied as context, and the LOD is ignoring it exactly per the
   pitfall in Module 1 Section 4.

## 6. Step 5: publish with certification and a data quality note

1. Publish the workbook to the "Northwind Analytics" project (Level 2
   Module 9/10), and mark the dashboard's underlying data source
   dependency as the certified `Orders` source from Module 8 rather than
   an embedded/local copy, so future viewers see the trust badge and any
   data quality warnings that source carries.

## Cheat sheet — full verification table

| Check | Expected value |
|---|---|
| Grand total (admin view) | 6880 |
| Alice (East) total | 2210 |
| Bob (West) total | 3750 |
| Carol (Central) total | 920 |
| East top category / share | Furniture / 97.3% |
| West top category / share | Electronics / 70.7% |
| Central top category / share | Furniture / 87.0% |
| RLS filter type | Context filter (so FIXED LODs respect it) |

## Exercise

A new manager, `frank@co.com`, is granted access to both East and West in
`RegionAccess`. Hand-compute what Frank's dashboard should show: total
Sales 2210 + 3750 = **5960**, and — since his RLS scope now spans two
regions — his `Category Sales in Region` FIXED calc should return **two**
Region×Category top rows (East/Furniture 2150, West/Electronics 2650), not
a single blended one, because the LOD is still fixed at the Region+Category
grain regardless of how many regions RLS allows through.
