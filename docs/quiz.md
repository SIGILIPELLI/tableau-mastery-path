---
title: "Final Quiz"
description: "Test what you have learned across all four levels of Tableau Mastery Path with a 20-question final quiz."
---

# Final Quiz

20 questions drawn from all four levels of this course. You need **70% (14 out of 20)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "What does this print for a FIXED-style total versus a view-level average?\n```python\nrows=[('E',100),('E',300),('W',200)]\nfrom collections import defaultdict\nt=defaultdict(int)\nfor r,s in rows: t[r]+=s\nprint(dict(t), round(sum(t.values())/len(t),1), round(sum(s for _,s in rows)/len(rows),1))\n```\n",
    "options": [
      "{'E': 400, 'W': 200} 300.0 200.0",
      "{'E': 400, 'W': 200} 300.0 300.0",
      "{'E': 200, 'W': 200} 300.0 200.0",
      "{'E': 400, 'W': 200} 200.0 300.0"
    ],
    "correctIndex": 0,
    "explanation": "The average of region totals differs from the average of rows because they aggregate at different levels. (A Python model of Tableau's calculation semantics; Tableau itself was not run.) Verified by running it."
  },
  {
    "question": "What does this table-calculation model print?\n```python\nfrom itertools import accumulate\nv=[5,10,15,20]\nprint(list(accumulate(v)), [round(100*x/sum(v),1) for x in v], [None]+[v[i]-v[i-1] for i in range(1,4)])\n```\n",
    "options": [
      "[5, 10, 15, 20] [10.0, 20.0, 30.0, 40.0] [None, 5, 5, 5]",
      "[5, 15, 30, 50] [25.0, 25.0, 25.0, 25.0] [None, 5, 5, 5]",
      "[5, 15, 30, 50] [10.0, 20.0, 30.0, 40.0] [5, 5, 5, 5]",
      "[5, 15, 30, 50] [10.0, 20.0, 30.0, 40.0] [None, 5, 5, 5]"
    ],
    "correctIndex": 3,
    "explanation": "Running sum, percent of total and difference are table calculations along the addressing direction. (A Python model of Tableau's calculation semantics; Tableau itself was not run.) Verified by running it."
  },
  {
    "question": "What does competition ranking give for [30, 10, 30, 20]?\n```python\nv=[30,10,30,20]\nprint([1+sum(1 for y in v if y>x) for x in v])\n```\n",
    "options": [
      "[2, 4, 2, 3]",
      "[1, 3, 2, 4]",
      "[1, 2, 1, 3]",
      "[1, 4, 1, 3]"
    ],
    "correctIndex": 3,
    "explanation": "Ties share a rank and the next rank is skipped; dense ranking would not skip. (A Python model of Tableau's calculation semantics; Tableau itself was not run.) Verified by running it."
  },
  {
    "question": "A join of orders to a customer table with a duplicated key. What happens to totals?\n```python\norders=[('c1',100),('c2',200)]\ncust=[('c1','R'),('c2','C'),('c2','C')]\nj=[(o,c) for o in orders for c in cust if o[0]==c[0]]\nprint(len(j), sum(o[1] for o,_ in j), sum(o[1] for o in orders))\n```\n",
    "options": [
      "2 500 300",
      "3 300 300",
      "2 300 300",
      "3 500 300"
    ],
    "correctIndex": 3,
    "explanation": "Duplicate keys fan out rows and inflate SUM; relationships and deduplication avoid it. (A Python model of Tableau's calculation semantics; Tableau itself was not run.) Verified by running it."
  },
  {
    "question": "A row-level security filter maps users to regions. What does ana (East, North) see?\n```python\norders=[('E',1),('W',2),('N',3),('E',4)]\nent={'ana':{'E','N'}}\nprint([o for o in orders if o[0] in ent.get('ana',set())], [o for o in orders if o[0] in ent.get('dan',set())])\n```\n",
    "options": [
      "[('E', 1), ('N', 3)] []",
      "[('E', 1), ('W', 2), ('N', 3), ('E', 4)] []",
      "[('E', 1), ('N', 3), ('E', 4)] []",
      "[('E', 1), ('N', 3), ('E', 4)] [('W', 2)]"
    ],
    "correctIndex": 2,
    "explanation": "A user with no entitlement should see nothing (fail closed). (A Python model of Tableau's calculation semantics; Tableau itself was not run.) Verified by running it."
  },
  {
    "question": "Which order of operations is right for these Tableau filters, earliest first?",
    "options": [
      "Extract filters, data source filters, context filters, FIXED LODs, dimension filters, INCLUDE/EXCLUDE LODs, measure filters, table calculations",
      "Dimension filters before context filters",
      "Table calculations first",
      "FIXED LODs last"
    ],
    "correctIndex": 0,
    "explanation": "Context filters make a dimension filter run before FIXED LODs."
  },
  {
    "question": "Why do table calculations change when you add a dimension to the view?",
    "options": [
      "They run on the marks in the view, so partitioning and addressing change with the layout",
      "They use the database",
      "They ignore filters",
      "They are cached"
    ],
    "correctIndex": 0,
    "explanation": "Check 'Compute using'."
  },
  {
    "question": "When should you choose a relationship over a join?",
    "options": [
      "Never",
      "For unions",
      "When tables have different grains, so each aggregates at its own level and avoids fan-out",
      "Always"
    ],
    "correctIndex": 2,
    "explanation": "Relationships are evaluated at query time."
  },
  {
    "question": "What is the main caution with data blending?",
    "options": [
      "Speed",
      "Colour",
      "The secondary source is aggregated by the linking fields and may mismatch the primary's grain",
      "Law"
    ],
    "correctIndex": 2,
    "explanation": "Prefer relationships where possible."
  },
  {
    "question": "What is the safest place to implement row-level security?",
    "options": [
      "A dashboard filter",
      "The data source (user filters, entitlement tables or data policies), not just a dashboard filter",
      "A title",
      "A tooltip"
    ],
    "correctIndex": 1,
    "explanation": "Dashboard filters can be edited or bypassed."
  },
  {
    "question": "Why show the extract refresh time on a dashboard?",
    "options": [
      "Law",
      "Speed",
      "Style",
      "Users need to know how fresh the numbers are to trust them"
    ],
    "correctIndex": 3,
    "explanation": "Transparency builds trust."
  },
  {
    "question": "What most often makes a dashboard slow?",
    "options": [
      "Colour",
      "Too many marks, sheets, quick filters and heavy calculations or custom SQL",
      "Titles",
      "Fonts"
    ],
    "correctIndex": 1,
    "explanation": "Use the Performance Recorder."
  },
  {
    "question": "What is a sound way to speed up a large dashboard?",
    "options": [
      "Live always",
      "Add filters",
      "Use an aggregated extract, reduce marks and filters and push work to the database",
      "More sheets"
    ],
    "correctIndex": 2,
    "explanation": "Measure each change."
  },
  {
    "question": "Which chart best shows a trend over time?",
    "options": [
      "Treemap",
      "A line chart with a continuous date axis",
      "Pie chart",
      "Packed bubbles"
    ],
    "correctIndex": 1,
    "explanation": "Bars work for discrete periods."
  },
  {
    "question": "What makes a dashboard effective for executives?",
    "options": [
      "Every metric",
      "A few KPIs with comparisons to target and clear drill-downs that answer specific questions",
      "Dense tables",
      "Many colours"
    ],
    "correctIndex": 1,
    "explanation": "Design backwards from decisions."
  },
  {
    "question": "What does certifying a data source tell users?",
    "options": [
      "It is new",
      "It has been reviewed and is trusted for use",
      "It is fast",
      "It is large"
    ],
    "correctIndex": 1,
    "explanation": "Part of governance."
  },
  {
    "question": "Why must you never publish sensitive data to Tableau Public?",
    "options": [
      "It is encrypted",
      "It is paid",
      "It is slow",
      "Everything published there is visible to anyone"
    ],
    "correctIndex": 3,
    "explanation": "Use Server or Cloud with permissions."
  },
  {
    "question": "What is the safest way to embed a Tableau view in an app?",
    "options": [
      "A shared password",
      "A public link",
      "Embedding API with a connected app (JWT) so each user's permissions apply",
      "A screenshot"
    ],
    "correctIndex": 2,
    "explanation": "Never share credentials."
  },
  {
    "question": "What does a BI Center of Excellence mainly do?",
    "options": [
      "Owns all data",
      "Sets standards and enables users through training, templates and community",
      "Builds every dashboard",
      "Replaces IT"
    ],
    "correctIndex": 1,
    "explanation": "Enablement beats control."
  },
  {
    "question": "What shows that a BI program works?",
    "options": [
      "Licences sold",
      "Dashboard count",
      "Adoption, trusted metrics and decisions improved by data, not the number of dashboards built",
      "Hours worked"
    ],
    "correctIndex": 2,
    "explanation": "Measure outcomes."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### LOD-style totals and table calculations

Model Tableau's calculation layers in Python on a small sales table: a FIXED total per region, percent of the region total, a cell's share of its month (EXCLUDE-style), then running sum, month-over-month change, rank and a moving average along the months. This teaches the semantics; Tableau itself was not run and you should rebuild these as real calculated fields in Tableau.

**Starter code**

```python
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

**Expected output**

```text
FIXED region: {'East': 3500, 'West': 2400}
percent of region total (orders 1-3): [34.3, 14.3, 18.8]
region share of month: {('2024-01', 'East'): 79.1, ('2024-01', 'West'): 20.9, ('2024-02', 'East'): 40.0, ('2024-02', 'West'): 60.0, ('2024-03', 'East'): 57.1, ('2024-03', 'West'): 42.9}
East [1700, 800, 1000] [1700, 2500, 3500] [None, -900, 200]
West [450, 1200, 750] [450, 1650, 2400] [None, 750, -450]
rank of months by total sales: {'2024-01': 1, '2024-02': 2, '2024-03': 3}
3-month moving average: [2075.0, 1966.7, 1875.0]
```

<details><summary>Show solution and explanation</summary>

<pre><code>from collections import defaultdict
from itertools import accumulate

# (order_id, month, region, category, sales)
rows = [
    (1, &quot;2024-01&quot;, &quot;East&quot;, &quot;Furniture&quot;, 1200), (2, &quot;2024-01&quot;, &quot;East&quot;, &quot;Tech&quot;, 500),
    (3, &quot;2024-01&quot;, &quot;West&quot;, &quot;Tech&quot;, 450),      (4, &quot;2024-02&quot;, &quot;East&quot;, &quot;Furniture&quot;, 800),
    (5, &quot;2024-02&quot;, &quot;West&quot;, &quot;Furniture&quot;, 300), (6, &quot;2024-02&quot;, &quot;West&quot;, &quot;Tech&quot;, 900),
    (7, &quot;2024-03&quot;, &quot;East&quot;, &quot;Tech&quot;, 600),      (8, &quot;2024-03&quot;, &quot;West&quot;, &quot;Tech&quot;, 750),
    (9, &quot;2024-03&quot;, &quot;East&quot;, &quot;Furniture&quot;, 400),
]

def fixed(dim_idx, measure_idx=4):
    &quot;&quot;&quot;{FIXED [dim]: SUM([measure])}: one total per dimension value, independent of the view.&quot;&quot;&quot;
    t = defaultdict(int)
    for r in rows:
        t[r[dim_idx]] += r[measure_idx]
    return dict(t)

region_total = fixed(2)
print(&quot;FIXED region:&quot;, region_total)

print(&quot;percent of region total (orders 1-3):&quot;,
      [round(100 * r[4] / region_total[r[2]], 1) for r in rows[:3]])

# EXCLUDE-style: percent of the month&#x27;s total while the view shows region x month
by_month = fixed(1)
cell = defaultdict(int)
for r in rows:
    cell[(r[1], r[2])] += r[4]
print(&quot;region share of month:&quot;, {k: round(100 * v / by_month[k[0]], 1) for k, v in sorted(cell.items())})

# table calcs along months for each region: running sum, month-over-month change and a rank
months = sorted({r[1] for r in rows})
for region in sorted(region_total):
    series = [cell.get((m, region), 0) for m in months]
    running = list(accumulate(series))
    change = [None] + [series[i] - series[i - 1] for i in range(1, len(series))]
    print(region, series, running, change)

month_rank = {m: 1 + sum(1 for x in by_month.values() if x &gt; by_month[m]) for m in months}
print(&quot;rank of months by total sales:&quot;, month_rank)
avg = [round(sum(v[max(0, i - 1):i + 2]) / len(v[max(0, i - 1):i + 2]), 1) for v in [[by_month[m] for m in months]] for i in range(len(months))]
print(&quot;3-month moving average:&quot;, avg)
</code></pre>

<p>A FIXED value is computed at the named level and then reused on every row; table calculations run afterwards on the aggregated marks along the addressing direction, which is why their results depend on the view. Verified by running the Python solution.</p>
</details>

### Row-level security and join fan-out

Model an entitlement-table row-level security rule and show how joining to a customer table with a duplicated key multiplies rows and inflates totals, then fix it by aggregating on a deduplicated table. It is a Python model of the logic behind user filters and relationships; Tableau Server and Cloud were not involved.

**Starter code**

```python
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

**Expected output**

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

<details><summary>Show solution and explanation</summary>

<pre><code># Row-level security and join fan-out, modelled in plain Python (Tableau itself is not involved)
orders = [
    (&quot;o1&quot;, &quot;East&quot;, &quot;c1&quot;, 100), (&quot;o2&quot;, &quot;East&quot;, &quot;c2&quot;, 250), (&quot;o3&quot;, &quot;West&quot;, &quot;c1&quot;, 80),
    (&quot;o4&quot;, &quot;West&quot;, &quot;c3&quot;, 400), (&quot;o5&quot;, &quot;North&quot;, &quot;c4&quot;, 60), (&quot;o6&quot;, &quot;North&quot;, &quot;c2&quot;, 140),
]
entitlements = [(&quot;ana&quot;, &quot;East&quot;), (&quot;ana&quot;, &quot;North&quot;), (&quot;bo&quot;, &quot;West&quot;), (&quot;cy&quot;, &quot;*&quot;)]
customers = [(&quot;c1&quot;, &quot;Retail&quot;), (&quot;c2&quot;, &quot;Corporate&quot;), (&quot;c2&quot;, &quot;Corporate&quot;), (&quot;c3&quot;, &quot;Retail&quot;), (&quot;c4&quot;, &quot;Home&quot;)]  # c2 duplicated by mistake

def visible(user):
    regions = {r for u, r in entitlements if u == user}
    if &quot;*&quot; in regions:
        return orders
    return [o for o in orders if o[1] in regions]

for user in [&quot;ana&quot;, &quot;bo&quot;, &quot;cy&quot;, &quot;dan&quot;]:
    rows = visible(user)
    print(f&quot;{user:4} sees {len(rows)} orders, total {sum(o[3] for o in rows)}&quot;)

# a join on a duplicated key multiplies rows (fan-out) and inflates SUM
joined = [(o, c) for o in orders for c in customers if o[2] == c[0]]
print(&quot;orders:&quot;, len(orders), &quot;joined rows:&quot;, len(joined))
print(&quot;true total:&quot;, sum(o[3] for o in orders), &quot;inflated total:&quot;, sum(o[3] for o, _ in joined))

# a relationship-style query aggregates each table at its own grain first, so duplicates do not inflate sales
distinct_customers = {c for c in customers}
by_segment = {}
for o in orders:
    seg = next((s for cid, s in distinct_customers if cid == o[2]), &quot;Unknown&quot;)
    by_segment[seg] = by_segment.get(seg, 0) + o[3]
print(&quot;by segment (deduplicated):&quot;, dict(sorted(by_segment.items())))
print(&quot;duplicate keys:&quot;, sorted({c[0] for c in customers if customers.count(c) &gt; 1}))
</code></pre>

<p>Security fails closed (users without entitlements see nothing, wildcard users see all). The duplicated customer key turns 6 orders into 8 joined rows and inflates the sales total from 1030 to 1420, the classic fan-out that relationships avoid. Verified by running the Python solution.</p>
</details>
