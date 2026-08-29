# 09 · Maps & Geographic Data

Tableau recognizes common geographic fields automatically and can plot them
on a built-in map with no extra setup. This module extends `Orders` with a
State column and builds a filled map of Sales by state.

## 1. Extending the dataset with geography

For this module, assume the `Orders` table (Module 1) has one more column,
**State**, assigning each order to a US state within its region:

| Order ID | Region | State | Sales |
|---|---|---|---|
| 1001 | East | New York | 1200 |
| 1002 | West | California | 450 |
| 1003 | East | New York | 60 |
| 1004 | Central | Texas | 800 |
| 1005 | West | California | 2200 |
| 1006 | East | Massachusetts | 950 |
| 1007 | Central | Texas | 120 |
| 1008 | West | Washington | 1100 |

## 2. Geographic roles

1. In the Data pane, a field named "State" (or "Country", "City", "Zip
   Code", and similar common names) automatically gets a small **globe
   icon**, meaning Tableau assigned it a **Geographic Role** — here,
   **State/Province**.
2. If a field isn't auto-detected (e.g. it's named `Location` instead of
   `State`), right-click it → **Geographic Role** → pick the correct role
   manually (State/Province, Country/Region, City, ZIP Code, etc.) so
   Tableau knows how to map its values to latitude/longitude.
3. Misspelled or ambiguous values (e.g. "Calif." instead of "California")
   show up as an **unrecognized** exclamation-mark icon on the field or a
   small "N unmatched" indicator — click it to manually match ambiguous
   values to the correct real-world location.

## 3. Building a filled map

1. Double-click **State** in the Data pane (or drag it to the canvas
   directly) — Tableau auto-generates **Longitude** and **Latitude**
   fields onto Columns and Rows and switches the mark type to **Map**, with
   one dot per state.
2. Drag **Sales** onto **Color** on the Marks card, and change the mark
   type dropdown from **Automatic**/Circle to **Map** → **Filled Map** (or
   use Show Me and pick the filled-map thumbnail) so each state renders as a
   shaded polygon rather than a dot.
3. Verify by hand: California's total Sales = 450 + 2200 = 2650 (highest,
   darkest shade), New York's = 1200 + 60 = 1260, Texas's = 800 + 120 = 920,
   Massachusetts's = 950, Washington's = 1100 — California should render as
   the darkest state on the color legend's high end.

## 4. Dual-axis maps (points over a filled background)

1. A common pattern layers detail points on top of a filled region map:
   duplicate the Latitude field (drag a second copy of Latitude onto Rows,
   next to the first), giving two overlapping map layers you can style
   independently.
2. Right-click the second **Latitude** pill → **Dual Axis**. Set the first
   layer's mark type to **Filled Map** (colored by, say, Region) and the
   second to **Circle** sized/colored by Sales — producing a background of
   regional shading with individual order markers on top.
3. This layered technique is the standard way to combine a "where" (filled
   regions) and a "how much, precisely, at this point" (sized circles) story
   in one map.

## 5. Limitations to know

1. Tableau's built-in geocoding covers standard administrative levels
   (country, state, county, city, ZIP) — a custom territory (e.g. an
   internal "Sales Territory 7") needs either a **custom geocoding** import
   (Map menu → Geocoding → Import Custom Geocoding) or joining to a
   reference table that maps it to a recognized geography.
2. A filled map's shading always represents an **aggregate over the whole
   region** (e.g. a whole state) — it cannot show sub-state variation
   without a finer geographic field (county, ZIP) actually present in the
   data.

## Cheat sheet

| Task | How |
|---|---|
| Confirm/set a geographic role | Right-click field → Geographic Role |
| Fix unmatched location values | Click the unmatched-values indicator on the field |
| Quick map from a geo field | Double-click the field in the Data pane |
| Switch dot map to filled/shaded | Marks card dropdown → Map → Filled Map |
| Layer points over a filled map | Duplicate Latitude → Dual Axis |
| Map a custom territory | Map menu → Geocoding → Import Custom Geocoding |

## Exercise

Using the extended `Orders` table with State (Section 1), build the filled
map of Sales by state described in Section 3, then add a second, circle
layer sized by SUM(Quantity) using the dual-axis technique from Section 4.
By hand, compute total Quantity per state from the original `Orders` table
(Module 1) matched to each state above, and confirm which state's circle
should render largest.
