# WCM Take-Home — Task 1: Service Guide & Service Details

This exercise reflects real maintenance work on our public site, where the **Service Guide** (a grid of service cards) and the **Service Details** page are rendered from WCM. You'll make one small UI change and one larger structural change to existing templates and components, then review a flawed snippet.

## How to work on this

1. **Fork** this repository and clone your fork.
2. Create a branch, e.g. `solution/<your-name>`.
3. Write your answers directly in this file under each **➡️ Your answer** heading (or add files under a `/solution` folder and link them from here).
4. Put all markup in fenced code blocks and **comment your code**.
5. Commit your work and **open a pull request against your own fork** (or send us the link to your fork).

**No WCM environment is required** — this is a paper exercise. We care about your design judgment and hand-written markup, not a running deployment.

**Time box:** ~2–3 hours. If you run short, prioritize Change Request 1 and the Code Review, and leave notes on the rest.

**On syntax:** exact WCM tag syntax varies by version. We're assessing whether your **structure, context handling, and intent** are correct.

**Follow-up:** a ~20-minute call where you walk us through your decisions.

---

## Current State (what exists today)

Names below are representative — adopt or improve the conventions as you see fit, and note any changes you make.

**Authoring template — `AT_Service`**

| Element | Type | Purpose |
|---|---|---|
| `serviceTitle` | Short text | Service name |
| `serviceSummary` | Text | One-line description shown on the card |
| `serviceBody` | Rich text | Full details on the detail page |
| `serviceSteps` | Rich text | How to use the service |
| `serviceIcon` | Image | Card icon |
| `serviceLink` | Link | Deep link to the actual service/transaction |

Services are classified with a **taxonomy `Service Categories`** (categories: `Individuals`, `Business`, `Vehicles`, `Travel`, `Documents`) applied via the content's profile.

**Service Guide** is rendered by a **Menu component `MENU_ServiceGuide`**: queries content of type `AT_Service`, scoped to the Services site area, sorted by title, output as a flat card grid via its result design.

**Service Details** is rendered by a **presentation template `PT_ServiceDetails`** bound to `AT_Service`. Simplified current markup:

```
<article class="service-detail">
  <h1>[Element context="current" type="content" key="serviceTitle"]</h1>
  <div class="summary">[Element context="current" type="content" key="serviceSummary"]</div>
  <div class="body">[Element context="current" type="content" key="serviceBody"]</div>
  <h2>How to use</h2>
  <div class="steps">[Element context="current" type="content" key="serviceSteps"]</div>
  <a class="cta" href="[Element context="current" type="content" key="serviceLink"]">Start service</a>
</article>
```

---

## Change Request 1 — small UI change

Product wants each service to optionally show a **status badge** — `New` or `Updated` — on both the **card** (Service Guide) and the **detail page**. Services with no status show no badge.

Deliver:

1. The change to `AT_Service` (which element type, and why).
2. The updated **card result design** markup in `MENU_ServiceGuide` so the badge renders per service, only when set.
3. The updated `PT_ServiceDetails` markup showing the badge.

Keep the badge text/style maintainable across both places.

### ➡️ Your answer

```
<!-- 1. AT_Service change: -->

<!-- 2. MENU_ServiceGuide card result design: -->

<!-- 3. PT_ServiceDetails: -->
```

_Rationale:_

---

## Change Request 2 — structural change

The flat alphabetical grid no longer scales. Restructure the **Service Guide** so that:

- Services are shown **grouped into sections by category** (`Individuals`, `Business`, …), each section with its own heading.
- A **"Featured services"** strip appears at the top, showing only services flagged as featured (in their natural priority order).

Deliver:

1. Any authoring-template/taxonomy changes needed to drive "featured" and the grouping.
2. Your component approach — one menu, several menus, or another mechanism — with the reasoning and the markup for the key piece(s).
3. A note on how this affects caching/performance on a high-traffic landing page.

### ➡️ Your answer

```
```

_Rationale:_

---

## Code Review — what's wrong with this?

A teammate committed the Service Guide card output below. **Identify every problem, rewrite it correctly, and state the impact of each issue.**

```
[Component name="ServiceCardStyles"]
<div style="display:flex; flex-wrap:wrap;">
  <!-- MENU_ServiceGuide config: type=AT_Service, scope=ALL libraries,
       max results=0 (unlimited), caching=none, sort=none -->
  [Component name="MENU_ServiceGuide"]
</div>

<!-- result design inside MENU_ServiceGuide: -->
<div class="card" style="width:300px; border:1px solid #ccc; margin:8px; padding:12px;">
  <img src="http://auth-cms-int.local:10039/wps/wcm/connect/services/icon.png" />
  <h3>[Property context="current" field="title"]</h3>
  <p>[Element context="current" type="content" key="serviceSummary"]</p>
  <a href="/wps/myportal/details">Read more</a>
</div>
```

### ➡️ Your answer

_Issues found (with impact):_

_Corrected markup:_

```
```

---

## Trade-off Questions

1. For the category-grouped layout in CR2, compare doing it with **one menu vs. several category-scoped menus vs. the WCM REST API + custom rendering**. What drives your choice, and what does each cost you?
2. The Service Guide is high-traffic; services change infrequently but a newly added service must appear **quickly**. Describe your caching and/or pre-render strategy and how you'd invalidate it on publish.

### ➡️ Your answer

_1._

_2._
