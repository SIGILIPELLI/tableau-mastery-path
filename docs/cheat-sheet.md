---
title: "Tableau Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free Tableau cheat sheet: 35 quick-reference tables from every level of the Tableau Mastery Path, each linked to its full lesson."
---

# Tableau Cheat Sheet

Quick-reference tables from **35 lessons** of the free Tableau Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Entry

### [01 · What Is Tableau?](level-1/01-what-is-tableau.md)

| Term | Meaning |
|---|---|
| Tableau Desktop | Authoring app — builds workbooks (.twb/.twbx) |
| Tableau Public | Free authoring, but every save publishes publicly |
| Tableau Server | Self-hosted publishing/sharing platform |
| Tableau Cloud | Vendor-hosted publishing/sharing platform |
| Tableau Prep | Separate app for cleaning/reshaping data pre-analysis |
| Dimension | Categorical field (slices the view); default blue/discrete |
| Measure | Numeric field (gets aggregated); default green/continuous |
| Data pane | Left-side list of every field in the connected data |
| Shelf | Drop target for fields (Columns, Rows, Filters, etc.) |
| Marks card | Controls mark type + Color/Size/Label/Detail/Tooltip |
| Live connection | Queries the source fresh every time |
| Extract | Snapshot of data saved into the workbook (.hyper) |

### [02 · Connecting to Data Sources](level-1/02-connecting-data-sources.md)

| Concept | Where / How |
|---|---|
| Connect to a file | Start screen → "To a File" → Excel/Text/JSON/PDF |
| Connect to a database | Start screen → "To a Server" → pick connector |
| Shape data pre-analysis | Data Source page (tables canvas + row preview) |
| Fix a column's type | Right-click header in preview → Change Data Type |
| Toggle Live/Extract | Top-right of Data Source page or worksheet |
| Refresh an extract | Data menu → Extract → Refresh |
| Combine multiple tables | Drag a second table onto Data Source canvas |

### [03 · Building Your First Visualization](level-1/03-first-visualization.md)

| Action | How |
|---|---|
| Add a dimension to slice the view | Drag to Columns or Rows |
| Add/aggregate a measure | Drag to Columns or Rows (defaults to SUM) |
| Change chart type quickly | Show Me panel (top right) |
| Change chart type manually | Marks card dropdown (Automatic/Bar/Line/…) |
| Split bars by a second dimension | Drag dimension onto Color |
| Show values on marks | Drag measure onto Label |
| Change a measure's aggregation | Right-click field on shelf → Measure |
| See exact value | Hover for tooltip |

### [04 · Chart Types &amp; When to Use Them](level-1/04-chart-types.md)

| Question being answered | Chart type | Key shelf setup |
|---|---|---|
| Compare a measure across categories | Bar | Dimension on Columns, Measure on Rows |
| Trend over time | Line | Continuous date on Columns, Measure on Rows |
| Relationship between two measures | Scatter | Measure on Columns, Measure on Rows, dimension on Detail |
| Share of a whole (few categories) | Pie | Dimension on Color, Measure on Angle |

### [05 · Building Dashboards Basics](level-1/05-dashboards-basics.md)

| Task | Where |
|---|---|
| Create a dashboard | Bottom tab bar → New Dashboard icon |
| Add a sheet to the dashboard | Drag from Dashboard pane onto canvas |
| Add a title/text | Objects section → Text, drag onto canvas |
| Tiled vs Floating object | Right-click object border |
| Configure filter/highlight actions | Dashboard menu → Actions... |
| Global filter across all sheets | Right-click filter card → Apply to Worksheets |

### [06 · Calculated Fields Basics](level-1/06-calculated-fields-basics.md)

| Action | How |
|---|---|
| Create a calculated field | Data pane dropdown → Create Calculated Field... |
| Aggregate ratio (correct pattern) | `SUM([A]) / SUM([B])` |
| Row-level conditional field | `IF ... THEN ... ELSEIF ... ELSE ... END` |
| Edit an existing calculation | Double-click it in the Data pane |
| See what uses a calculated field | Right-click → Properties |
| Format as percentage | Right-click pill → Format... → Numbers → Percentage |

### [07 · Filters &amp; Parameters](level-1/07-filters-parameters.md)

| Action | How |
|---|---|
| Filter a dimension | Drag field to Filters shelf → pick values |
| Filter a measure | Drag field to Filters shelf → pick aggregation → range |
| Show a filter control | Right-click filter card → Show Filter |
| Make a context filter | Right-click filter card → Add to Context |
| Create a parameter | Data pane dropdown → Create Parameter... |
| Show a parameter control | Right-click parameter → Show Parameter Control |
| Reference a parameter in a formula | Use its name in brackets, e.g. `[Sales Threshold]` |

### [08 · Table Calculations Basics](level-1/08-table-calculations-basics.md)

| Table calc | Answers | Key setting |
|---|---|---|
| Running Total | Cumulative sum so far | Compute Using = direction to accumulate |
| Percent of Total | Share of the row/column/table | Compute Using = what "total" means |
| Rank | Position among values in view | Compute Using = direction to rank across |
| (general) | — | Always confirm Compute Using matches your intended grouping |

### [09 · Maps &amp; Geographic Data](level-1/09-maps-geographic-data.md)

| Task | How |
|---|---|
| Confirm/set a geographic role | Right-click field → Geographic Role |
| Fix unmatched location values | Click the unmatched-values indicator on the field |
| Quick map from a geo field | Double-click the field in the Data pane |
| Switch dot map to filled/shaded | Marks card dropdown → Map → Filled Map |
| Layer points over a filled map | Duplicate Latitude → Dual Axis |
| Map a custom territory | Map menu → Geocoding → Import Custom Geocoding |

### [10 · Project — Build a Sales Performance Dashboard](level-1/10-project-sales-performance-dashboard.md)

| Requirement | Verified by |
|---|---|
| 4 worksheets built with correct totals | Hand-computed totals in Step 1 |
| Dashboard assembled with title + 2×2 layout | Visual check |
| Global Region filter applied to all sheets | Filter control changes all 4 sheets |
| Filter action: Region bar → Time + Ratio charts | Click East, verify Step 4.2 |
| Highlight action: State map → Region bar (hover) | Hover California, verify Step 4.3 |

## Level 2 · Intermediate

### [01 · Advanced Calculated Fields](level-2/01-advanced-calculated-fields.md)

| Form | Behavior |
|---|---|
| `{FIXED [dim] : AGG}` | Computes at named dim(s) only, ignores view |
| `{INCLUDE [dim] : AGG}` | View detail + named dim, then re-aggregate |
| `{EXCLUDE [dim] : AGG}` | View detail minus named dim |
| Percent of fixed total | `SUM([X]) / {FIXED [dim]: SUM([X])}` |
| Nested IF | `IF...ELSEIF...ELSEIF...ELSE...END` |
| Month name | `DATENAME('month', [date])` |
| Quarter number | `DATEPART('quarter', [date])` |

### [02 · Advanced Dashboard Design &amp; Interactivity](level-2/02-advanced-dashboard-design.md)

| Feature | Where |
|---|---|
| Floating container | Layout pane → toggle Floating, or Shift-drag |
| Filter action | Dashboard → Actions → Add Action → Filter |
| Highlight action | Dashboard → Actions → Add Action → Highlight |
| URL action | Dashboard → Actions → Add Action → URL |
| Show/Hide button | Layout pane → Add Show/Hide Button |
| Phone layout | Dashboard → Device Layouts → Add Phone Layout |

### [03 · Data Blending vs Joins vs Relationships](level-2/03-blending-joins-relationships.md)

| Concept | Key trait |
|---|---|
| Join | Physical row combination; risk of fan-out duplication |
| Relationship | Default; aggregates each table at native detail first |
| Blend | For separate data sources; aggregate-only, one-directional |
| Fan-out risk | One-to-many join key duplicates the "one" side's measures |
| Sanity check | Grand total (6880) should survive any join/relationship |

### [04 · Advanced Table Calculations](level-2/04-advanced-table-calculations.md)

| Table calc | Formula concept | Verified value (example) |
|---|---|---|
| Running total (partitioned) | `RUNNING_SUM(SUM([Sales]))` | East final = 2210 |
| Moving average (trailing 2) | avg of current + 1 prior | Order 3 = 255 |
| Percent difference | `(cur − prev) / prev` | Order 5 = +175.0% |
| Rank (competition) | `RANK(SUM([Sales]))` | Desk = 1 (2300) |
| Addressing | Direction the calc scans | e.g. Order Date |
| Partitioning | Dimension that resets the calc | e.g. Region |

### [05 · Formatting &amp; Storytelling with Data](level-2/05-formatting-storytelling.md)

| Element | Rule of thumb |
|---|---|
| Currency formatting | 0 decimals unless sub-dollar precision matters |
| Percentage formatting | 1 decimal place is standard |
| Categorical color | Unordered dimensions (Region, Category) |
| Sequential color | Ordered magnitude, one direction (Sales) |
| Diverging color | Values that cross a meaningful zero (Profit) |
| Title | State the finding, not the field name |
| Story point | One narrative beat per point; short caption |

### [06 · Tableau Prep Basics](level-2/06-tableau-prep-basics.md)

| Task | Prep step |
|---|---|
| Fix inconsistent casing | Clean → Change Case |
| Strip currency symbols, fix type | Clean → change data type |
| Parse a text date | Clean → change type to Date + pattern |
| Merge near-duplicate values | Clean → Group Values |
| Pre-summarize rows | Aggregate step |
| Reshape wide→long | Pivot → Columns to Rows |
| Write cleaned result | Output → Save to file / Publish |

### [07 · Working with Multiple Data Sources](level-2/07-multiple-data-sources.md)

| Situation | Approach |
|---|---|
| Two DB types, need row-level combine | Cross-database join |
| Two sources, aggregate-only combine | Blend (Level 2 Module 3) |
| Two sources, no combine needed | Separate sheets, same dashboard |
| Swap environments (prod/staging) | Parameter-driven data source |
| Verify a dashboard didn't silently fail to filter | Click a mark, confirm expected behavior |

### [08 · Performance Optimization Basics](level-2/08-performance-optimization-basics.md)

| Technique | Effect |
|---|---|
| Extract (`.hyper`) | Local, pre-optimized copy vs. live re-query |
| Extract with aggregation | Collapses to only the dimensions used |
| Extract filter | Drops rows before the extract is built |
| Context filter | Runs first; can be cached; fixes calc-order bugs |
| Fewer marks / less granularity | Less rendering + query work |
| Performance Recording | Diagnoses where time is actually spent |

### [09 · Publishing to Tableau Public/Server](level-2/09-publishing-tableau-public-server.md)

| Task | Where |
|---|---|
| Publish (public, free) | Server → Tableau Public → Save to Tableau Public |
| Publish (org, licensed) | Server → Publish Workbook |
| Reusable data source | Publish data source separately from workbook |
| Set access | Project/workbook Permissions dialog |
| Keep data current | Extract refresh schedule |
| Recurring snapshot | Subscribe (self or others) |
| Threshold notification | Data-Driven Alert |

### [10 · Project — Multi-Source Executive Dashboard](level-2/10-project-executive-dashboard.md)

| Layer | Delivered |
|---|---|
| Data | Orders + Targets cross-database join, verified totals |
| KPIs | Total Sales, Total Profit, Overall Attainment |
| Charts | Region-vs-Target bars, Category breakdown, Tier detail |
| Interactivity | Filter action, Show/Hide container, phone layout |
| Formatting | Finding-based title, diverging Attainment color |
| Publish | Server publish, restricted download, refresh schedule |

## Level 3 · Advanced

### [01 · Advanced LOD Expressions](level-3/01-advanced-lod-expressions.md)

| Expression | Result for this dataset |
|---|---|
| `{FIXED [Region],[Category]: SUM(Sales)}` | 6 region×category cells |
| `{FIXED [Category]: AVG({FIXED [Region],[Category]: SUM(Sales)})}` | Furniture=1350 |
| FIXED + non-context filter | Ignores the filter |
| FIXED + context filter | Respects the filter |

### [02 · Tableau Server/Cloud Administration Basics](level-3/02-server-cloud-administration.md)

| Concept | Role |
|---|---|
| Gateway | Routes requests |
| VizQL Server | Renders views |
| Backgrounder | Runs refreshes/subscriptions |
| Repository | Postgres metadata store |
| Site | Isolated tenant within a Server/Cloud pod |
| Project | Folder + permission template |
| Site role | Viewer / Explorer / Creator ceiling |
| Group | Bulk permission management unit |

### [03 · Row-Level Security](level-3/03-row-level-security.md)

| Approach | Best for |
|---|---|
| `USERNAME()` + joined entitlement table | Per-user, data-driven access |
| `ISMEMBEROF('group')` | Access maps cleanly to Server groups |
| Filter shelf, set to True | Where the RLS calc is applied |
| Context filter | Performance + correct filter ordering |
| Zero-row test for unmapped user | Confirms fail-closed, not fail-open |

### [03 · Advanced Tableau Prep Workflows](level-3/04-advanced-prep-workflows.md)

| Step | Purpose | Verified output |
|---|---|---|
| Clean (Trim, Change Case) | Fix formatting inconsistency | 3 distinct Regions/Categories |
| Calculated field (parse currency) | Text → number | Shipping total 139.4 |
| Aggregate | Group by cleaned Region | East 2210, West 3750, Central 920 |
| Pivot | Rows → columns | Wide table, rows sum to Region totals |
| Output + incremental refresh | Efficient nightly load | Keyed on non-reused Order ID |

### [04 · Custom Visualizations &amp; Extensions Overview](level-3/05-custom-visualizations-extensions.md)

| API call | Returns |
|---|---|
| `getSummaryDataAsync()` | Aggregated rows as rendered (e.g. 3 Region rows) |
| `getUnderlyingDataAsync()` | Row-level data (up to 8 Orders rows) |
| `applyFilterAsync(field, values, type)` | Sets a filter programmatically |
| `clearFilterAsync(field)` | Removes a filter |

### [05 · Advanced Performance Tuning](level-3/06-advanced-performance-tuning.md)

| Lever | Effect | Cost order (cheapest → priciest per row) |
|---|---|---|
| Data source filter | Reduces rows pulled | Earliest, cheapest overall |
| Extract vs. live | Removes per-query source round-trip | Depends on source, test both |
| LOD FIXED | Computed once, cached | Low |
| Aggregate calc | Once per rendered group | Low–Medium |
| Row-level calc | Once per row | Medium |
| Table calc | Once per mark, needs full partition | High |
| Context filter | Materializes before other filters/LODs | Medium (recomputes on change) |

### [06 · Integrating Tableau with Databases at Scale](level-3/07-databases-at-scale.md)

| Pattern | Where the join runs | Best for |
|---|---|---|
| Native join (Data pane) | Tableau (live) or extract build | Small-to-medium dims joined to a fact |
| Custom SQL | Database | Complex logic, DB-side optimization |
| Cross-database join | Tableau only | Tables live in different systems |
| Left join with unmatched key | Tableau/DB per join location | Preserving orphan rows (nulls surface) |

### [07 · Governance &amp; Data Source Certification](level-3/08-governance-certification.md)

| Concept | What it does | What it does NOT do |
|---|---|---|
| Certification | Marks the recommended, trusted source | Doesn't block use of other sources |
| Data quality warning | Flags known issues (staleness, etc.) | Doesn't fix the underlying data |
| Lineage | Shows source → field → downstream use | Doesn't prevent breaking changes alone |
| Impact analysis | Shows what breaks before a change | Doesn't make the change safe automatically |

### [08 · Embedding Tableau Dashboards](level-3/09-embedding-dashboards.md)

| Task | API call/mechanism |
|---|---|
| Basic embed | `<tableau-viz src="..." toolbar="bottom">` |
| Authenticate host visitor | Connected apps (OAuth) or trusted ticket |
| Apply a filter from host page | `applyFilterAsync(field, values, 'replace')` |
| React to a click in the viz | `markselectionchanged` event |
| RLS inside an embed | Still enforced — same as direct Server access |

### [09 · Project — Enterprise Dashboard with Row-Level Security](level-3/10-project-enterprise-dashboard-rls.md)

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

## Level 4 · Master

### [01 · Enterprise Tableau Architecture](level-4/01-enterprise-architecture.md)

| Concern | Key architecture decision |
|---|---|
| Tenant isolation | Single site + strong permissions vs. multi-site |
| Process placement | Separate Gateway/VizQL from Backgrounder for large refresh volume |
| Resilience | HA (node redundancy) vs. DR (separate site + backup) |
| Sizing | Viewer count AND Backgrounder/refresh load, not just one |

### [01 · Building a BI Center of Excellence](level-4/02-bi-center-of-excellence.md)

| CoE function | Analogous course module |
|---|---|
| Governance | Level 3 Module 8 — certification |
| Enablement | Training authors in course techniques (LOD, RLS, Prep) |
| Platform | Level 4 Module 1 — architecture/capacity |
| Support | Resolving discrepancies like the 6880 vs. 6700 case |

### [03 · Tableau + Advanced Analytics Integration (R/Python via Tableau)](level-4/04-advanced-analytics-integration.md)

| Function | Returns |
|---|---|
| `SCRIPT_REAL` | Floating-point result per partition |
| `SCRIPT_STR` | String result |
| `SCRIPT_BOOL` | Boolean result |
| `SCRIPT_INT` | Integer result |
| `_argN` | Nth Tableau field/aggregation passed into the script |

### [04 · Scaling Tableau Server Deployments](level-4/05-scaling-server-deployments.md)

| Symptom | Likely bottleneck | Scaling response |
|---|---|---|
| Slow dashboard render, viewers complain during business hours | VizQL Server saturated | Add VizQL nodes (horizontal) or vertical scale |
| Refresh jobs run late/queue backs up overnight | Backgrounder saturated | Add Backgrounder capacity, stagger schedules |
| Metadata operations (publish, permissions) slow | Repository (Postgres) under load | Vertical scale Repository node, check for backup/vacuum conflicts |
| All processes fine individually, but content-level views slow | Calc complexity (LOD/table calc heavy) | Optimize workbook (Level 3 Module 5), not infrastructure |

### [07 · Cost &amp; License Management at Scale](level-4/08-cost-license-management.md)

| Lever | Mechanism | Where covered |
|---|---|---|
| Tier right-sizing | Match license to actual usage, not convenience | Section 2 |
| Usage-based downgrade | Admin Views activity history | Section 3 |
| Extract consolidation | Fewer duplicate sources = less refresh/storage cost | Section 4, Level 4 Module 2/3 |
| Quarterly review | Recurring cadence catches drift early | Section 5 |

---

Go deeper with the full [Tableau Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
