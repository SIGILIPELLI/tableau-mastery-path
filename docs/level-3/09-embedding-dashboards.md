---
description: "Embedding Tableau Dashboards — Embedding puts a Tableau view inside another web page or application. This module covers the Embedding API v3…"
---

# 08 · Embedding Tableau Dashboards

Embedding puts a Tableau view inside another web page or application. This
module covers the Embedding API v3, ticketed/trusted authentication, and
filter pass-through — verified against the `Orders` dashboard.

## 1. Basic embed with the JS Embedding API v3

1. Embedding a published view uses a custom element:

    ```html
    <tableau-viz id="ordersViz"
      src="https://server.example.com/views/Northwind/OrdersDashboard"
      toolbar="bottom">
    </tableau-viz>
    ```

2. This renders the same dashboard a Server user would see by navigating
   directly to it — including the same 8-row `Orders` data and 6880 grand
   total — inside the host page's DOM, subject to the viewer's own
   permissions on Server.

## 2. Authentication: how an embedded view resolves permissions

1. If the embedding host page's visitor is **not** already signed into
   Tableau Server/Cloud in that browser, the embed either prompts for
   Tableau credentials directly, or — in most production embeds — uses
   **connected apps** (OAuth-based trusted authentication, replacing the
   older ticket-based trusted auth) so the host application vouches for
   the user without a separate Tableau login prompt.
2. Whichever visitor is authenticated, **all of Level 3 Module 3's RLS
   still applies** — an embedded view is not a bypass of row-level
   security. If the authenticated user maps to `alice@co.com` in
   `RegionAccess`, the embedded dashboard still shows only the 3 East
   orders totaling 2210, exactly as it would viewing directly on Server.

## 3. Passing filters into an embedded view

1. The Embedding API lets the host page set filters programmatically:

    ```js
    const viz = document.getElementById('ordersViz');
    await viz.workbook.activeSheet.applyFilterAsync(
      'Region', ['West'], 'replace');
    ```

2. Verify: after this call, the embedded dashboard's Sales total should
   read 3750 (West's total from every prior module), not the full 6880 —
   if it still shows 6880, the filter call targeted the wrong sheet object
   or fired before the viz finished loading (a common embedding bug — wait
   for the `firstinteractive` event before calling filter methods).

## 4. Listening for events from the embedded view

1. The embed fires events the host page can react to, e.g.
   `viz.addEventListener('markselectionchanged', handler)` — a host
   application could use this to show a custom detail panel when a user
   clicks the East bar in a Region chart.
2. Hand-verify the event payload's data matches expectations: clicking the
   East bar should deliver mark data whose Sales value is 2210 (or, if the
   chart's granularity is per-order rather than per-region, the individual
   Order IDs/Sales for 1001, 1003, 1006) — a payload showing an unrelated
   region's numbers indicates a stale/cached viz reference in the host
   page's JavaScript.

## 5. Licensing and viewer counts for embedding

1. Every person who views an embedded dashboard still consumes a Tableau
   Server/Cloud viewer license (or falls under an embedded analytics
   licensing agreement, if the host application is customer-facing at
   scale) — embedding is a *delivery mechanism*, not a way to avoid
   licensing, a governance point that connects to Module 8's cost topics
   (Level 4 Module 8 covers license management at scale).

## How It Actually Works

1. `<tableau-viz>` is a native Web Component (a custom element registered
   by the Embedding API v3 script) — the browser instantiates it like any
   built-in tag, and internally it creates a sandboxed iframe pointed at
   the view URL. All JS API calls (`activeSheet`, `applyFilterAsync`)
   don't manipulate the DOM directly; they post structured messages across
   the iframe boundary to the actual Tableau rendering runtime running
   inside it, which is why calling a method before the iframe's content
   has loaded (before `firstinteractive` fires) silently no-ops or throws
   rather than queuing — there's no listener on the other side yet.
2. Connected apps authentication works by JWT exchange rather than a
   shared ticket: the host application, holding a registered connected
   app's secret, signs a short-lived JWT asserting the visitor's username
   and (optionally) group membership, and passes it to the embed. Tableau
   Server/Cloud validates the signature against the connected app's public
   key, then treats the request exactly as if that user had logged in
   directly — the same session, same group memberships, same everything —
   which is precisely why RLS (Level 3 Module 3) still evaluates against
   `RegionAccess` for that resolved username inside the embed; the embed
   never bypasses the permission-resolution pipeline, it just supplies the
   identity differently.
3. `applyFilterAsync` sends a filter-change message into the iframe that
   VizQL treats identically to a click on a filter card in the Server UI —
   it triggers the same query-recompilation path from Module 6, including
   re-running any live query or re-scanning the extract with the new
   `WHERE Region='West'` predicate. It does **not** bypass RLS's own
   predicate, because RLS is injected into the query at the data-source
   level (as its own always-on filter), so the effective query becomes the
   AND of both: the RLS predicate and the host-applied filter — which is
   exactly why Carol (Central-only) requesting `Region='East'` produces an
   empty result set rather than an error: `WHERE Region='Central' AND
   Region='East'` is satisfiable by zero rows, and VizQL renders that as a
   normal empty view, not a failure.
4. Every rendered embed still opens a real Tableau session server-side
   (visible in Server's Admin views the same as a direct browser session),
   which is the mechanism behind Section 5's licensing point — the license
   consumption check happens at session creation on the server, completely
   independent of whether the browser reaching that session is Tableau's
   own UI or a third-party host page.

## Cheat sheet

| Task | API call/mechanism |
|---|---|
| Basic embed | `<tableau-viz src="..." toolbar="bottom">` |
| Authenticate host visitor | Connected apps (OAuth) or trusted ticket |
| Apply a filter from host page | `applyFilterAsync(field, values, 'replace')` |
| React to a click in the viz | `markselectionchanged` event |
| RLS inside an embed | Still enforced — same as direct Server access |

## Exercise

A host page embeds the `Orders` dashboard for a visitor authenticated as
`carol@co.com` (Central-only access per Module 3's `RegionAccess` table),
then calls `applyFilterAsync('Region', ['East'], 'replace')`. Hand-reason
what the embedded view should show: RLS restricts Carol to Central rows
regardless of the filter call, and the filter additionally asks for East —
the intersection of "Central only" and "East only" is empty, so the
dashboard should render **zero rows**, not an error and not Central's 920.
