---
title: "Tableau Interview Questions and Answers (180 Questions)"
description: "180 Tableau interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the Tableau Mastery Path."
---

# Tableau Interview Questions and Answers

**180 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free Tableau Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Entry

### [01 · What Is Tableau?](level-1/01-what-is-tableau.md)

**Q1.** What does Tableau do?

**Answer:** Connects to data and lets you build interactive visualisations and dashboards by dragging fields

It is a visual analytics platform.

**Q2.** What is the difference between Tableau Desktop and Tableau Public?

**Answer:** Public is a free edition whose workbooks are published to the public web, so it is unsuitable for sensitive data

Never publish confidential data to Public.

**Q3.** What are the main Tableau products in a typical deployment?

**Answer:** Desktop for authoring, Server or Cloud for sharing, and Prep for data preparation

Plus Bridge for private data on Cloud.

**Q4.** What does 'visual analytics' emphasise?

**Answer:** Asking and answering questions interactively through visual exploration

Flow of analysis.

### [02 · Connecting to Data Sources](level-1/02-connecting-data-sources.md)

**Q5.** What is a Tableau extract (.hyper)?

**Answer:** A compressed snapshot of data stored in Tableau's Hyper engine for fast queries

Refreshed on a schedule.

**Q6.** When is a live connection preferable?

**Answer:** When data changes constantly and the source database can handle the query load

Extracts help when sources are slow.

**Q7.** What is a data source filter?

**Answer:** A filter applied at the data source that restricts rows before they reach any worksheet

Applied early in the order of operations.

**Q8.** What does Data Interpreter help with?

**Answer:** Cleaning messy Excel sheets by detecting headers, footers and sub-tables

Review the result.

### [03 · Building Your First Visualization](level-1/03-first-visualization.md)

**Q9.** What colours distinguish dimensions and measures in the data pane?

**Answer:** Blue for dimensions and green for measures (by default)

Dimensions are categories; measures are numbers.

**Q10.** What is the difference between discrete and continuous fields?

**Answer:** Discrete fields create headers (blue pills); continuous fields create axes (green pills)

Colour of the pill matters.

**Q11.** What does dragging a date to Columns do by default?

**Answer:** Creates a discrete year header (you can change the date part or make it continuous)

Right-click to choose.

**Q12.** What does the Marks card control?

**Answer:** Colour, size, label, detail, tooltip and shape of marks

Detail adds granularity without visual encoding.

### [04 · Chart Types &amp; When to Use Them](level-1/04-chart-types.md)

**Q13.** Which chart suits a trend over time?

**Answer:** A line chart

Continuous date on the axis.

**Q14.** Which chart suits comparing categories?

**Answer:** A bar chart

Sorted bars read easily.

**Q15.** When is a scatter plot appropriate?

**Answer:** To show the relationship between two measures

Add trend lines.

**Q16.** Why are pie charts often discouraged?

**Answer:** Humans compare angles poorly, so bars communicate better except with very few slices

Use sparingly.

### [05 · Building Dashboards Basics](level-1/05-dashboards-basics.md)

**Q17.** What is a dashboard in Tableau?

**Answer:** A collection of sheets, objects and filters arranged on one canvas

Add interactivity with actions.

**Q18.** What do dashboard actions allow?

**Answer:** Filter, highlight, go to URL or sheet, change a parameter or set values from clicks

Make dashboards interactive.

**Q19.** Why use layout containers?

**Answer:** They arrange objects horizontally or vertically so the dashboard resizes predictably

Tiled layouts are easier to maintain.

**Q20.** What is a good dashboard principle?

**Answer:** Show the most important view first, keep it uncluttered and answer a specific question

Design for the audience.

### [06 · Calculated Fields Basics](level-1/06-calculated-fields-basics.md)

**Q21.** A calculated field computes profit ratio as SUM(profit)/SUM(sales). What is the ratio for these rows (percent, 1 decimal)?

```python
rows = [
 ("East","Furniture",1200,180),("East","Furniture",800,60),("East","Tech",500,120),
 ("West","Tech",450,90),("West","Furniture",300,-20),("West","Tech",900,200),
 ("North","Office",150,30),("North","Tech",600,150),
]  # (region, category, sales, profit)
s=sum(r[2] for r in rows); p=sum(r[3] for r in rows)
print(round(100*p/s,1))
```

**Answer:** 16.5

A ratio of sums is not the average of row ratios. (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q22.** Why does the average of row-level ratios differ from the ratio of sums? (average of profit/sales per row, percent)

```python
rows = [
 ("East","Furniture",1200,180),("East","Furniture",800,60),("East","Tech",500,120),
 ("West","Tech",450,90),("West","Furniture",300,-20),("West","Tech",900,200),
 ("North","Office",150,30),("North","Tech",600,150),
]  # (region, category, sales, profit)
print(round(100*sum(r[3]/r[2] for r in rows)/len(rows),1))
```

**Answer:** 15.9

Aggregate first, then divide. (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q23.** What is the difference between a row-level and an aggregate calculation?

**Answer:** Row-level calculations run per row before aggregation; aggregate calculations run on aggregated values

You cannot mix aggregate and non-aggregate without care.

**Q24.** Which function handles NULLs gracefully?

**Answer:** IFNULL or ZN (zero if null)

Prevents blanks propagating.

### [07 · Filters &amp; Parameters](level-1/07-filters-parameters.md)

**Q25.** What is a parameter?

**Answer:** A workbook variable the user can change to drive calculations, filters or reference lines

Not tied to data values unless you configure it.

**Q26.** What is the difference between a filter and a parameter?

**Answer:** A filter removes rows from the view; a parameter is a value that calculations and visuals can use

Use a parameter in a calculated field to filter.

**Q27.** What is a context filter used for?

**Answer:** Making a filter run before others (for example before Top N or FIXED LODs depending on version)

Appears grey.

**Q28.** What is Top N filtering?

**Answer:** Showing only the N highest or lowest members by a measure

Beware of order of operations with other filters.

### [08 · Table Calculations Basics](level-1/08-table-calculations-basics.md)

**Q29.** A running total of sales by order: [100, 200, 50, 150]. What is the cumulative sequence?

```python
from itertools import accumulate
print(list(accumulate([100,200,50,150])))
```

**Answer:** [100, 300, 350, 500]

RUNNING_SUM does this along the addressing direction. (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q30.** Percent of total for sales [100, 200, 50, 150]?

```python
v=[100,200,50,150]
print([round(100*x/sum(v),1) for x in v])
```

**Answer:** [20.0, 40.0, 10.0, 30.0]

A quick table calculation. (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q31.** What do addressing and partitioning mean for table calculations?

**Answer:** Partitioning splits the data into groups; addressing is the direction calculated within each partition

Compute using Table (Across) etc.

**Q32.** When does a table calculation run?

**Answer:** After the query returns, on the aggregated data in the view

So it only sees marks in the view.

### [09 · Maps &amp; Geographic Data](level-1/09-maps-geographic-data.md)

**Q33.** What do Tableau maps need to place data?

**Answer:** Geographic roles on fields (country, state, city) or latitude and longitude

Check for unrecognised locations.

**Q34.** What is a filled map?

**Answer:** Shapes (polygons) coloured by a measure, such as states by sales

Choropleth.

**Q35.** What is a common pitfall of choropleth maps?

**Answer:** Large low-population areas dominate visually; consider rates or symbols

Normalise the measure.

**Q36.** How can you fix ambiguous location names?

**Answer:** Use higher-level fields such as Country and State, or custom geocoding

Edit locations.

### [10 · Project — Build a Sales Performance Dashboard](level-1/10-project-sales-performance-dashboard.md)

**Q37.** What is a good project structure for a sales dashboard?

**Answer:** KPIs at the top, trends, breakdowns and filters with clear titles and consistent colours

Answer business questions.

**Q38.** Which region has the largest total sales in the sample data?

```python
rows = [
 ("East","Furniture",1200,180),("East","Furniture",800,60),("East","Tech",500,120),
 ("West","Tech",450,90),("West","Furniture",300,-20),("West","Tech",900,200),
 ("North","Office",150,30),("North","Tech",600,150),
]  # (region, category, sales, profit)
from collections import Counter
c=Counter()
for r in rows: c[r[0]]+=r[2]
print(c.most_common(1)[0])
```

**Answer:** ('East', 2500)

Group by region and sum. (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q39.** How do you keep a dashboard performing well?

**Answer:** Limit marks and filters, use extracts and avoid heavy calculations

Measure with the performance recorder.

**Q40.** How do you share it?

**Answer:** Publish to Tableau Server, Cloud or Public (only for non-sensitive data)

Permissions matter.

## Level 2 · Intermediate

### [01 · Advanced Calculated Fields](level-2/01-advanced-calculated-fields.md)

**Q41.** A LOD-style calculation: average of region totals vs average of rows. What is each? (region totals average, row average)

```python
rows = [
 ("East","Furniture",1200,180),("East","Furniture",800,60),("East","Tech",500,120),
 ("West","Tech",450,90),("West","Furniture",300,-20),("West","Tech",900,200),
 ("North","Office",150,30),("North","Tech",600,150),
]  # (region, category, sales, profit)
from collections import defaultdict
t=defaultdict(int)
for r in rows: t[r[0]]+=r[2]
print(round(sum(t.values())/len(t),1), round(sum(r[2] for r in rows)/len(rows),1))
```

**Answer:** 1633.3 612.5

{FIXED [Region]: SUM([Sales])} gives a value per region regardless of the view's level. (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q42.** What does the IF/ELSEIF/END structure do?

**Answer:** Returns different values by condition in a calculated field

CASE is cleaner for one field.

**Q43.** What do string functions like SPLIT and CONTAINS help with?

**Answer:** Cleaning and extracting parts of text fields

Use before aggregation.

**Q44.** What does DATETRUNC do?

**Answer:** Truncates a date to a given level such as month, keeping it as a date

Different from DATEPART.

### [02 · Advanced Dashboard Design &amp; Interactivity](level-2/02-advanced-dashboard-design.md)

**Q45.** What is the key design question for any dashboard?

**Answer:** Who is the audience and what decision should it support?

Design backwards from decisions.

**Q46.** Why design for specific device sizes?

**Answer:** Layouts that fit desktops can be unreadable on phones; use device layouts

Fixed vs automatic sizing.

**Q47.** What helps consistent look and feel?

**Answer:** Shared colour palettes, fonts and templates across dashboards

Document standards.

**Q48.** What is progressive disclosure?

**Answer:** Show summary first and let users drill into detail on demand

Reduces clutter.

### [03 · Data Blending vs Joins vs Relationships](level-2/03-blending-joins-relationships.md)

**Q49.** What does a relationship do compared with a join?

**Answer:** It keeps tables separate and joins them at query time at the right level of detail

Avoids many fan-out problems.

**Q50.** What is data blending?

**Answer:** Combining data from separate published/connected sources by linking fields, aggregating the secondary source

Primary source drives the view.

**Q51.** An inner join of orders (3 rows with customer ids [1,1,2]) with customers (ids [1,2]) returns how many rows? And if customers had a duplicate id 1 row?

```python
orders=[1,1,2]
cust=[1,2]
cust2=[1,1,2]
print(sum(1 for o in orders for c in cust if o==c), sum(1 for o in orders for c in cust2 if o==c))
```

**Answer:** 3 5

Duplicate keys multiply rows: the fan-out problem. (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q52.** What is a union?

**Answer:** Stacking rows from tables with the same columns

Useful for monthly files.

### [04 · Advanced Table Calculations](level-2/04-advanced-table-calculations.md)

**Q53.** What is a moving average table calc used for?

**Answer:** Smoothing noisy time series using a window of neighbouring marks

WINDOW_AVG with offsets.

**Q54.** 3-point moving average of [10, 20, 30, 40, 50] (window of previous, current, next; edges use available points)?

```python
v=[10,20,30,40,50]
out=[]
for i in range(len(v)):
    w=v[max(0,i-1):i+2]
    out.append(sum(w)/len(w))
print(out)
```

**Answer:** [15.0, 20.0, 30.0, 40.0, 45.0]

WINDOW_AVG(SUM([Sales]), -1, 1). (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q55.** Rank of values [30, 10, 30, 20] with competition ranking (ties share a rank, next skips)?

```python
v=[30,10,30,20]
print([1+sum(1 for y in v if y>x) for x in v])
```

**Answer:** [1, 4, 1, 3]

RANK(SUM(x)) works like this; DENSE would give [1,3,1,2]. (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q56.** Why do table calculations depend on the layout of the view?

**Answer:** They operate on the marks present, so changing dimensions changes partitions and results

Check the compute-using settings.

### [05 · Formatting &amp; Storytelling with Data](level-2/05-formatting-storytelling.md)

**Q57.** What makes a story or narrative dashboard effective?

**Answer:** A clear takeaway in titles, annotation of key points and a logical flow

Guide the reader.

**Q58.** How should colour be used?

**Answer:** Sparingly and consistently to highlight meaning, with accessible contrast and colour-blind-safe palettes

Avoid red-green contrast alone.

**Q59.** What do annotations add?

**Answer:** Context that explains why a mark matters

Short and specific.

**Q60.** What is the data-ink idea (Tufte)?

**Answer:** Maximise the share of ink that conveys data and remove chartjunk

Simplify.

### [06 · Tableau Prep Basics](level-2/06-tableau-prep-basics.md)

**Q61.** What is Tableau Prep for?

**Answer:** Visually cleaning, shaping and combining data before analysis

Flows produce outputs.

**Q62.** What does a pivot step do?

**Answer:** Turns columns into rows (or rows into columns)

Fix wide data.

**Q63.** What is an aggregate step?

**Answer:** Groups rows and computes summaries

Changes granularity.

**Q64.** Why schedule Prep flows?

**Answer:** Keep prepared data fresh without manual runs (Prep Conductor)

Needs Data Management.

### [07 · Working with Multiple Data Sources](level-2/07-multiple-data-sources.md)

**Q65.** When would you use multiple data sources in one workbook?

**Answer:** When questions need data from different systems that cannot be combined in one connection

Use relationships or blends carefully.

**Q66.** What is cross-database join?

**Answer:** Joining tables from different connections in the data source tab

Can be slower than same-database joins.

**Q67.** What is the risk of blending on different granularity?

**Answer:** Aggregating mismatched levels can give misleading numbers

Know the grain.

**Q68.** How do you verify a combined data source?

**Answer:** Check row counts and totals against the source systems

Reconcile.

### [08 · Performance Optimization Basics](level-2/08-performance-optimization-basics.md)

**Q69.** What does the Performance Recorder show?

**Answer:** Where time goes when a workbook loads or interacts (queries, layout, rendering)

Start there.

**Q70.** Which tactic usually speeds up a slow dashboard?

**Answer:** Use extracts, reduce marks and the number of sheets and filters

Simplify.

**Q71.** Why can many quick filters slow a dashboard?

**Answer:** Each filter triggers queries to populate its values

Use 'Only relevant values' sparingly or parameters.

**Q72.** What does aggregating an extract do?

**Answer:** Stores data at a higher level so queries touch fewer rows

Fits dashboards that don't need row detail.

### [09 · Publishing to Tableau Public/Server](level-2/09-publishing-tableau-public-server.md)

**Q73.** What is Tableau Public suitable for?

**Answer:** Public portfolios and open data, since everything published is visible to anyone

Never publish sensitive data.

**Q74.** What do permissions on Server or Cloud control?

**Answer:** Who can view, edit and download content in projects

Grant to groups.

**Q75.** What is a project on Tableau Server/Cloud?

**Answer:** A folder that organises content and sets permissions

Use hierarchies.

**Q76.** What should you do before publishing a workbook?

**Answer:** Remove unused fields, test on the target, check data sources and permissions

Quality check.

### [10 · Project — Multi-Source Executive Dashboard](level-2/10-project-executive-dashboard.md)

**Q77.** What does an executive dashboard need?

**Answer:** A few KPIs, clear trends and drill-downs answering key business questions at a glance

Focus on decisions.

**Q78.** What should KPIs show?

**Answer:** Value, comparison to target or prior period and direction

Context makes numbers meaningful.

**Q79.** How do you handle data refresh expectations?

**Answer:** Show last refreshed time and schedule extracts appropriately

Trust requires transparency.

**Q80.** What is a good way to test a dashboard with users?

**Answer:** Observe them completing real tasks and ask what decisions they make

Iterate.

## Level 3 · Advanced

### [01 · Advanced LOD Expressions](level-3/01-advanced-lod-expressions.md)

**Q81.** FIXED vs INCLUDE vs EXCLUDE: what does {FIXED [Region]: SUM([Sales])} return per row, and the percent of region total for the first row?

```python
rows = [
 ("East","Furniture",1200,180),("East","Furniture",800,60),("East","Tech",500,120),
 ("West","Tech",450,90),("West","Furniture",300,-20),("West","Tech",900,200),
 ("North","Office",150,30),("North","Tech",600,150),
]  # (region, category, sales, profit)
from collections import defaultdict
t=defaultdict(int)
for r in rows: t[r[0]]+=r[2]
r0=rows[0]
print(t[r0[0]], round(100*r0[2]/t[r0[0]],1))
```

**Answer:** 2500 48.0

FIXED computes at the stated dimension regardless of the view, so each row can be compared to its region total. (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q82.** Where do FIXED LODs sit in Tableau's order of operations?

**Answer:** After context filters and before dimension filters

Dimension filters don't affect FIXED unless in context.

**Q83.** What does INCLUDE do?

**Answer:** Computes at the view's level plus the extra dimension(s) given

Then aggregates to the view.

**Q84.** What does EXCLUDE do?

**Answer:** Computes at the view's level minus the specified dimension(s)

Useful for percent of total.

### [02 · Tableau Server/Cloud Administration Basics](level-3/02-server-cloud-administration.md)

**Q85.** What does a Tableau Server administrator manage?

**Answer:** Users, groups, sites, licences, schedules and server resources

Cloud is managed by Tableau.

**Q86.** What is a site on Server or Cloud?

**Answer:** A separate area with its own users, projects and content

Multi-tenancy.

**Q87.** What is Tableau Bridge for?

**Answer:** Keeping Cloud content connected to data behind a firewall

Refresh private data.

**Q88.** Why use groups for permissions?

**Answer:** Easier administration than assigning individuals

Sync from identity provider.

### [03 · Row-Level Security](level-3/03-row-level-security.md)

**Q89.** A row-level security rule: users see rows where Region matches their entitlement. Which regions can 'ana' (entitled to East and North) see in the sample, and how many rows?

```python
rows = [
 ("East","Furniture",1200,180),("East","Furniture",800,60),("East","Tech",500,120),
 ("West","Tech",450,90),("West","Furniture",300,-20),("West","Tech",900,200),
 ("North","Office",150,30),("North","Tech",600,150),
]  # (region, category, sales, profit)
ent={'ana':{'East','North'}}
vis=[r for r in rows if r[0] in ent['ana']]
print(sorted({r[0] for r in vis}), len(vis))
```

**Answer:** ['East', 'North'] 5

A user filter joins each user to the entitled values. (Checked with a Python model of the calculation; Tableau itself was not used.)

**Q90.** Which Tableau function returns the current user?

**Answer:** USERNAME() (or FULLNAME())

Use with an entitlement table.

**Q91.** Why prefer data-source-level RLS to a filter on a dashboard?

**Answer:** Filters can be removed by editors or bypassed by downloads; data source rules apply regardless

Secure at the source.

**Q92.** What must you test with RLS?

**Answer:** Each role's view, including edge cases such as users with no entitlements

Use impersonation.

### [03 · Advanced Tableau Prep Workflows](level-3/04-advanced-prep-workflows.md)

**Q93.** What is a Prep flow's main benefit over ad hoc cleaning?

**Answer:** Repeatable, documented and schedulable steps

Version control matters too.

**Q94.** What does a join step with a mismatch show?

**Answer:** Join results by category (matched, left-only, right-only) so you can inspect them

Check before output.

**Q95.** What is an incremental refresh?

**Answer:** Appending only new rows instead of reloading everything

Faster for large data.

**Q96.** What output options does Prep provide?

**Answer:** Files (hyper, CSV), databases and published data sources

Pick for the consumer.

### [04 · Custom Visualizations &amp; Extensions Overview](level-3/05-custom-visualizations-extensions.md)

**Q97.** What are Tableau extensions?

**Answer:** Web apps that add functionality to dashboards, using the Extensions API

Review their permissions.

**Q98.** When use a custom visual (such as a Sankey) over a standard chart?

**Answer:** When the standard chart cannot show the relationship and the audience can read it

Complexity has a cost.

**Q99.** What is a risk of third-party extensions?

**Answer:** They can access dashboard data, so vet them and restrict allowed extensions

Admin controls.

**Q100.** What is a viz extension?

**Answer:** A custom mark type added to the Marks card

Newer capability.

### [05 · Advanced Performance Tuning](level-3/06-advanced-performance-tuning.md)

**Q101.** What do context filters, extracts and fewer marks do?

**Answer:** They reduce the work per query and speed up rendering

Measure each change.

**Q102.** Why can custom SQL hurt performance?

**Answer:** The database cannot optimise it as well and it runs as a subquery for every query

Use views or tables.

**Q103.** What does Tableau Server's Resource Monitoring Tool help with?

**Answer:** Finding bottlenecks in server resources and slow content

Proactive tuning.

**Q104.** What should you tune first?

**Answer:** The slowest, most used workbooks identified by data

Measure.

### [06 · Integrating Tableau with Databases at Scale](level-3/07-databases-at-scale.md)

**Q105.** What is a good practice for large databases and Tableau?

**Answer:** Push work to the database with indexes, aggregates and extracts of aggregated data

Mind query cost.

**Q106.** What does initial SQL do?

**Answer:** Runs statements when a connection opens, such as setting a role

Use with care.

**Q107.** Why use materialised or aggregate tables for dashboards?

**Answer:** Pre-computation makes queries fast and predictable

Match the grain.

**Q108.** What does query folding mean here?

**Answer:** The tool pushes operations down to the source database

Reduces transferred data.

### [07 · Governance &amp; Data Source Certification](level-3/08-governance-certification.md)

**Q109.** What does certifying a data source signal?

**Answer:** It is trusted, reviewed and recommended for use

Governance.

**Q110.** What is a data quality warning?

**Answer:** A visible notice on content when data is stale or under review

Transparency.

**Q111.** What does a governance model define?

**Answer:** Roles, standards, review processes and who can publish what

Balance control and agility.

**Q112.** What is content lifecycle management?

**Answer:** Reviewing, archiving and retiring unused or outdated content

Reduces clutter.

### [08 · Embedding Tableau Dashboards](level-3/09-embedding-dashboards.md)

**Q113.** What is the safest way to embed a dashboard in an app?

**Answer:** Use the Embedding API with a connected app (JWT) so users get appropriate access

Respect permissions.

**Q114.** What does the Embedding API v3 provide?

**Answer:** A web component for embedding views with events and filters

JavaScript.

**Q115.** Why not embed with shared credentials?

**Answer:** Everyone would see the same data and auditing is lost

Use per-user authentication.

**Q116.** What must you consider for embedded performance?

**Answer:** Load time, caching and the size of the view

Test in the host app.

### [09 · Project — Enterprise Dashboard with Row-Level Security](level-3/10-project-enterprise-dashboard-rls.md)

**Q117.** What does the enterprise dashboard with RLS combine?

**Answer:** Governed data sources, row-level security, performance tuning and certified content

Level 3 capstone.

**Q118.** How do you demonstrate RLS works?

**Answer:** Test as different users and show that each sees only entitled data

Document tests.

**Q119.** What should the entitlement table contain?

**Answer:** Users or groups mapped to the values they may see

Maintain it centrally.

**Q120.** What is a common RLS pitfall?

**Answer:** Users with no mapping seeing everything or nothing unexpectedly

Define defaults.

## Level 4 · Master

### [01 · Enterprise Tableau Architecture](level-4/01-enterprise-architecture.md)

**Q121.** What does an enterprise BI architecture include?

**Answer:** Data platform, governed data sources, Tableau deployment, security and support processes

Beyond a single tool.

**Q122.** Why separate environments (dev, test, prod)?

**Answer:** To validate changes before users see them

Promotion workflows.

**Q123.** What is a hub-and-spoke analytics model?

**Answer:** Central governed data with decentralised departmental analysis

Balance control and agility.

**Q124.** What belongs in an architecture decision?

**Answer:** Requirements, options, trade-offs and ownership

Record it.

### [01 · Building a BI Center of Excellence](level-4/02-bi-center-of-excellence.md)

**Q125.** What is a BI Center of Excellence?

**Answer:** A team that sets standards, enables users and promotes data culture

Training, governance, support.

**Q126.** What does a CoE do for adoption?

**Answer:** Provides training, templates, office hours and a community

Enablement beats control.

**Q127.** How do you measure CoE success?

**Answer:** Adoption, trust, time to insight and business outcomes

Outcomes over output.

**Q128.** Who should sponsor a CoE?

**Answer:** Executive leadership tied to business value

Visible sponsorship.

### [02 · Advanced Governance &amp; Content Strategy](level-4/03-advanced-governance-content-strategy.md)

**Q129.** What does a content strategy decide?

**Answer:** What is certified, who owns it and how content is organised and retired

Reduces duplication.

**Q130.** What is a certified vs sandbox space?

**Answer:** Certified is governed and trusted; sandbox is for exploration

Label clearly.

**Q131.** How do you prevent metric sprawl?

**Answer:** Publish shared certified metrics and definitions

One source of truth.

**Q132.** Why document data definitions?

**Answer:** So everyone interprets metrics the same way

A data dictionary.

### [03 · Tableau + Advanced Analytics Integration (R/Python via Tableau)](level-4/04-advanced-analytics-integration.md)

**Q133.** How can Tableau integrate with Python or R?

**Answer:** Through TabPy or Rserve for calculated fields calling models

Mind security and performance.

**Q134.** What is a risk of external scripts in calculations?

**Answer:** Latency and dependency on an external service

Cache results where possible.

**Q135.** What does Einstein Discovery add (Tableau Pulse etc.)?

**Answer:** Predictions and insights surfaced in dashboards and metrics

Explain results.

**Q136.** When should analytics be precomputed?

**Answer:** When models are heavy; write results to a table and visualise them

Dashboards stay fast.

### [04 · Scaling Tableau Server Deployments](level-4/05-scaling-server-deployments.md)

**Q137.** How do you scale a Tableau Server deployment?

**Answer:** Add nodes and distribute processes (VizQL, backgrounder, data engine) by workload

Monitor first.

**Q138.** What is high availability on Server?

**Answer:** Multiple nodes and a failover-ready repository and coordination service

Test failover.

**Q139.** What does a backgrounder do?

**Answer:** Runs scheduled extract refreshes and subscriptions

Scale with refresh load.

**Q140.** Why plan capacity with load testing?

**Answer:** To know how many users and refreshes the deployment supports

Tabjolt and similar tools.

### [05 · Data Strategy &amp; Self-Service BI Enablement](level-4/06-data-strategy-self-service-bi.md)

**Q141.** What is self-service BI's main risk?

**Answer:** Inconsistent metrics and ungoverned data without guardrails

Balance with governance.

**Q142.** What enables safe self-service?

**Answer:** Certified data sources, clear definitions and training

Guardrails.

**Q143.** What does a data strategy align?

**Answer:** Data, technology and people with business goals

Start from outcomes.

**Q144.** How do you drive adoption?

**Answer:** Solve real problems, train users and show quick wins

Champions help.

### [06 · Advanced Embedding &amp; Tableau Extensions API Overview](level-4/07-advanced-embedding-extensions-api.md)

**Q145.** What can the Tableau REST API do?

**Answer:** Automate administration such as publishing, permissions and refreshes

Script governance.

**Q146.** What does the Metadata API expose?

**Answer:** Lineage and metadata about content, fields and data sources

Impact analysis.

**Q147.** How do you authenticate automation safely?

**Answer:** Personal access tokens or connected apps with least privilege, stored securely

Rotate tokens.

**Q148.** What is webhooks' use?

**Answer:** Triggering actions when events such as refresh failures occur

Integrate with alerts.

### [07 · Cost &amp; License Management at Scale](level-4/08-cost-license-management.md)

**Q149.** What drives Tableau licence cost?

**Answer:** Role types (Creator, Explorer, Viewer) and deployment size

Right-size roles.

**Q150.** How can you reduce licence waste?

**Answer:** Review usage, reassign inactive users and match roles to needs

Use admin insights.

**Q151.** What other costs matter?

**Answer:** Infrastructure, data platform, training and support

Total cost of ownership.

**Q152.** How do you show value from BI?

**Answer:** Link dashboards to decisions and measurable outcomes

ROI stories.

### [08 · Career Growth: Analyst to BI Leader](level-4/09-career-growth-analyst-to-bi-leader.md)

**Q153.** What skills move an analyst toward BI leadership?

**Answer:** Data storytelling, stakeholder management, governance and mentoring

Beyond technical skill.

**Q154.** What builds a credible portfolio?

**Answer:** Real, well-documented projects showing questions, method and impact (without confidential data)

Public datasets help.

**Q155.** Why mentor others?

**Answer:** It deepens your understanding and multiplies your impact

Leaders grow people.

**Q156.** What is a sound way to plan career growth?

**Answer:** Set goals, seek feedback and fill gaps with projects and learning

Track progress.

### [09 · Capstone — Full BI Program Design](level-4/10-capstone-full-bi-program-design.md)

**Q157.** What does the capstone BI program design cover?

**Answer:** Strategy, governance, architecture, enablement, adoption and measurement

A complete operating model.

**Q158.** What should the roadmap show?

**Answer:** Phases, priorities, owners and success measures

Deliver value early.

**Q159.** How do you handle risks?

**Answer:** Identify, prioritise and mitigate or accept them explicitly

Risk register.

**Q160.** What is the strongest evidence of a successful program?

**Answer:** Adoption, trusted metrics and decisions improved by data

Outcomes.

## Mixed questions across all levels

**Q161.** What does this print for a FIXED-style total versus a view-level average?

```python
rows=[('E',100),('E',300),('W',200)]
from collections import defaultdict
t=defaultdict(int)
for r,s in rows: t[r]+=s
print(dict(t), round(sum(t.values())/len(t),1), round(sum(s for _,s in rows)/len(rows),1))
```

**Answer:** {'E': 400, 'W': 200} 300.0 200.0

The average of region totals differs from the average of rows because they aggregate at different levels. (A Python model of Tableau's calculation semantics; Tableau itself was not used.) The Python was run and its output checked.

**Q162.** What does this table-calculation model print?

```python
from itertools import accumulate
v=[5,10,15,20]
print(list(accumulate(v)), [round(100*x/sum(v),1) for x in v], [None]+[v[i]-v[i-1] for i in range(1,4)])
```

**Answer:** [5, 15, 30, 50] [10.0, 20.0, 30.0, 40.0] [None, 5, 5, 5]

Running sum, percent of total and difference are table calculations along the addressing direction. (A Python model of Tableau's calculation semantics; Tableau itself was not used.) The Python was run and its output checked.

**Q163.** What does competition ranking give for [30, 10, 30, 20]?

```python
v=[30,10,30,20]
print([1+sum(1 for y in v if y>x) for x in v])
```

**Answer:** [1, 4, 1, 3]

Ties share a rank and the next rank is skipped; dense ranking would not skip. (A Python model of Tableau's calculation semantics; Tableau itself was not used.) The Python was run and its output checked.

**Q164.** A join of orders to a customer table with a duplicated key. What happens to totals?

```python
orders=[('c1',100),('c2',200)]
cust=[('c1','R'),('c2','C'),('c2','C')]
j=[(o,c) for o in orders for c in cust if o[0]==c[0]]
print(len(j), sum(o[1] for o,_ in j), sum(o[1] for o in orders))
```

**Answer:** 3 500 300

Duplicate keys fan out rows and inflate SUM; relationships and deduplication avoid it. (A Python model of Tableau's calculation semantics; Tableau itself was not used.) The Python was run and its output checked.

**Q165.** A row-level security filter maps users to regions. What does ana (East, North) see?

```python
orders=[('E',1),('W',2),('N',3),('E',4)]
ent={'ana':{'E','N'}}
print([o for o in orders if o[0] in ent.get('ana',set())], [o for o in orders if o[0] in ent.get('dan',set())])
```

**Answer:** [('E', 1), ('N', 3), ('E', 4)] []

A user with no entitlement should see nothing (fail closed). (A Python model of Tableau's calculation semantics; Tableau itself was not used.) The Python was run and its output checked.

**Q166.** Which order of operations is right for these Tableau filters, earliest first?

**Answer:** Extract filters, data source filters, context filters, FIXED LODs, dimension filters, INCLUDE/EXCLUDE LODs, measure filters, table calculations

Context filters make a dimension filter run before FIXED LODs.

**Q167.** Why do table calculations change when you add a dimension to the view?

**Answer:** They run on the marks in the view, so partitioning and addressing change with the layout

Check 'Compute using'.

**Q168.** When should you choose a relationship over a join?

**Answer:** When tables have different grains, so each aggregates at its own level and avoids fan-out

Relationships are evaluated at query time.

**Q169.** What is the main caution with data blending?

**Answer:** The secondary source is aggregated by the linking fields and may mismatch the primary's grain

Prefer relationships where possible.

**Q170.** What is the safest place to implement row-level security?

**Answer:** The data source (user filters, entitlement tables or data policies), not just a dashboard filter

Dashboard filters can be edited or bypassed.

**Q171.** Why show the extract refresh time on a dashboard?

**Answer:** Users need to know how fresh the numbers are to trust them

Transparency builds trust.

**Q172.** What most often makes a dashboard slow?

**Answer:** Too many marks, sheets, quick filters and heavy calculations or custom SQL

Use the Performance Recorder.

**Q173.** What is a sound way to speed up a large dashboard?

**Answer:** Use an aggregated extract, reduce marks and filters and push work to the database

Measure each change.

**Q174.** Which chart best shows a trend over time?

**Answer:** A line chart with a continuous date axis

Bars work for discrete periods.

**Q175.** What makes a dashboard effective for executives?

**Answer:** A few KPIs with comparisons to target and clear drill-downs that answer specific questions

Design backwards from decisions.

**Q176.** What does certifying a data source tell users?

**Answer:** It has been reviewed and is trusted for use

Part of governance.

**Q177.** Why must you never publish sensitive data to Tableau Public?

**Answer:** Everything published there is visible to anyone

Use Server or Cloud with permissions.

**Q178.** What is the safest way to embed a Tableau view in an app?

**Answer:** Embedding API with a connected app (JWT) so each user's permissions apply

Never share credentials.

**Q179.** What does a BI Center of Excellence mainly do?

**Answer:** Sets standards and enables users through training, templates and community

Enablement beats control.

**Q180.** What shows that a BI program works?

**Answer:** Adoption, trusted metrics and decisions improved by data, not the number of dashboards built

Measure outcomes.

## Practical coding questions

### LOD-style totals and table calculations

Model Tableau's calculation layers in Python on a small sales table: a FIXED total per region, percent of the region total, a cell's share of its month (EXCLUDE-style), then running sum, month-over-month change, rank and a moving average along the months. This teaches the semantics; Tableau itself was not run and you should rebuild these as real calculated fields in Tableau.

Starter code:

```
from collections import defaultdict
from itertools import accumulate

# (order_id, month, region, category, sales)
rows = [
    (1, "2024-01", "East", "Furniture", 1200), (2, "2024-01", "East", "Tech", 500),
    (3, "2024-01", "West", "Tech", 450),      (4, "2024-02", "East", "Furniture", 800),
    (5, "2024-02", "West", "Furniture", 300), (6, "2024-02", "West", "Tech", 900),
    (7, "2024-03", "East", "Tech", 600),      (8, "2024-03", "West", "Tech", 750),
    (9, "2024-03", "East", "Furniture", 400),
]

# TODO: fixed(dim_idx, measure_idx=4)   {FIXED [dim]: SUM([measure])} as a dict, independent of what the "view" shows
# Then reproduce, in this order:
#   FIXED region totals, percent of region total for the first three orders (1 decimal),
#   the share of each (month, region) cell in its month's total (an EXCLUDE-style calculation),
#   per region over the sorted months: the series, its running sum and the month-over-month change (None for the first month),
#   the rank of each month by total sales (competition ranking), and a 3-month moving average (previous/current/next, fewer at the edges, 1 decimal)
```

Expected output:

```text
FIXED region: {'East': 3500, 'West': 2400}
percent of region total (orders 1-3): [34.3, 14.3, 18.8]
region share of month: {('2024-01', 'East'): 79.1, ('2024-01', 'West'): 20.9, ('2024-02', 'East'): 40.0, ('2024-02', 'West'): 60.0, ('2024-03', 'East'): 57.1, ('2024-03', 'West'): 42.9}
East [1700, 800, 1000] [1700, 2500, 3500] [None, -900, 200]
West [450, 1200, 750] [450, 1650, 2400] [None, 750, -450]
rank of months by total sales: {'2024-01': 1, '2024-02': 2, '2024-03': 3}
3-month moving average: [2075.0, 1966.7, 1875.0]
```

??? success "Show a solution"

    ```
    from collections import defaultdict
    from itertools import accumulate

    # (order_id, month, region, category, sales)
    rows = [
        (1, "2024-01", "East", "Furniture", 1200), (2, "2024-01", "East", "Tech", 500),
        (3, "2024-01", "West", "Tech", 450),      (4, "2024-02", "East", "Furniture", 800),
        (5, "2024-02", "West", "Furniture", 300), (6, "2024-02", "West", "Tech", 900),
        (7, "2024-03", "East", "Tech", 600),      (8, "2024-03", "West", "Tech", 750),
        (9, "2024-03", "East", "Furniture", 400),
    ]

    def fixed(dim_idx, measure_idx=4):
        """{FIXED [dim]: SUM([measure])}: one total per dimension value, independent of the view."""
        t = defaultdict(int)
        for r in rows:
            t[r[dim_idx]] += r[measure_idx]
        return dict(t)

    region_total = fixed(2)
    print("FIXED region:", region_total)

    print("percent of region total (orders 1-3):",
          [round(100 * r[4] / region_total[r[2]], 1) for r in rows[:3]])

    # EXCLUDE-style: percent of the month's total while the view shows region x month
    by_month = fixed(1)
    cell = defaultdict(int)
    for r in rows:
        cell[(r[1], r[2])] += r[4]
    print("region share of month:", {k: round(100 * v / by_month[k[0]], 1) for k, v in sorted(cell.items())})

    # table calcs along months for each region: running sum, month-over-month change and a rank
    months = sorted({r[1] for r in rows})
    for region in sorted(region_total):
        series = [cell.get((m, region), 0) for m in months]
        running = list(accumulate(series))
        change = [None] + [series[i] - series[i - 1] for i in range(1, len(series))]
        print(region, series, running, change)

    month_rank = {m: 1 + sum(1 for x in by_month.values() if x > by_month[m]) for m in months}
    print("rank of months by total sales:", month_rank)
    avg = [round(sum(v[max(0, i - 1):i + 2]) / len(v[max(0, i - 1):i + 2]), 1) for v in [[by_month[m] for m in months]] for i in range(len(months))]
    print("3-month moving average:", avg)
    ```

    A FIXED value is computed at the named level and then reused on every row; table calculations run afterwards on the aggregated marks along the addressing direction, which is why their results depend on the view. Verified by running the Python solution.

### Row-level security and join fan-out

Model an entitlement-table row-level security rule and show how joining to a customer table with a duplicated key multiplies rows and inflates totals, then fix it by aggregating on a deduplicated table. It is a Python model of the logic behind user filters and relationships; Tableau Server and Cloud were not involved.

Starter code:

```
# Row-level security and join fan-out, modelled in plain Python (Tableau itself is not involved)
orders = [
    ("o1", "East", "c1", 100), ("o2", "East", "c2", 250), ("o3", "West", "c1", 80),
    ("o4", "West", "c3", 400), ("o5", "North", "c4", 60), ("o6", "North", "c2", 140),
]
entitlements = [("ana", "East"), ("ana", "North"), ("bo", "West"), ("cy", "*")]
customers = [("c1", "Retail"), ("c2", "Corporate"), ("c2", "Corporate"), ("c3", "Retail"), ("c4", "Home")]  # c2 duplicated by mistake

# TODO: visible(user)  rows the user may see: the entitled regions, everything for "*", nothing for unknown users
# TODO: print each user's order count and total; show the fan-out of joining orders to the customer list that contains a duplicate key
#       (joined rows, true total and inflated total); then aggregate by segment using the DEDUPLICATED customer list, and list duplicate keys
# Expected shape: "ana  sees 4 orders, total 550" ... "orders: 6 joined rows: 8" ...
```

Expected output:

```text
ana  sees 4 orders, total 550
bo   sees 2 orders, total 480
cy   sees 6 orders, total 1030
dan  sees 0 orders, total 0
orders: 6 joined rows: 8
true total: 1030 inflated total: 1420
by segment (deduplicated): {'Corporate': 390, 'Home': 60, 'Retail': 580}
duplicate keys: ['c2']
```

??? success "Show a solution"

    ```
    # Row-level security and join fan-out, modelled in plain Python (Tableau itself is not involved)
    orders = [
        ("o1", "East", "c1", 100), ("o2", "East", "c2", 250), ("o3", "West", "c1", 80),
        ("o4", "West", "c3", 400), ("o5", "North", "c4", 60), ("o6", "North", "c2", 140),
    ]
    entitlements = [("ana", "East"), ("ana", "North"), ("bo", "West"), ("cy", "*")]
    customers = [("c1", "Retail"), ("c2", "Corporate"), ("c2", "Corporate"), ("c3", "Retail"), ("c4", "Home")]  # c2 duplicated by mistake

    def visible(user):
        regions = {r for u, r in entitlements if u == user}
        if "*" in regions:
            return orders
        return [o for o in orders if o[1] in regions]

    for user in ["ana", "bo", "cy", "dan"]:
        rows = visible(user)
        print(f"{user:4} sees {len(rows)} orders, total {sum(o[3] for o in rows)}")

    # a join on a duplicated key multiplies rows (fan-out) and inflates SUM
    joined = [(o, c) for o in orders for c in customers if o[2] == c[0]]
    print("orders:", len(orders), "joined rows:", len(joined))
    print("true total:", sum(o[3] for o in orders), "inflated total:", sum(o[3] for o, _ in joined))

    # a relationship-style query aggregates each table at its own grain first, so duplicates do not inflate sales
    distinct_customers = {c for c in customers}
    by_segment = {}
    for o in orders:
        seg = next((s for cid, s in distinct_customers if cid == o[2]), "Unknown")
        by_segment[seg] = by_segment.get(seg, 0) + o[3]
    print("by segment (deduplicated):", dict(sorted(by_segment.items())))
    print("duplicate keys:", sorted({c[0] for c in customers if customers.count(c) > 1}))
    ```

    Security fails closed (users without entitlements see nothing, wildcard users see all). The duplicated customer key turns 6 orders into 8 joined rows and inflates the sales total from 1030 to 1420, the classic fan-out that relationships avoid. Verified by running the Python solution.

---

More free practice: the [Tableau Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
