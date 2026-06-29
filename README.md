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
For this change, I would add an optional selection field to AT_Service. 
Name: serviceStatus / type: Option Selection / Select list 
Allowed values: 
 - New 
 - Updated 

I would use a controlled select field instead of free text so authors cannot enter inconsistent values like "new", "NEW", "recent", or "updated service". 
Services with no selected value would show no badge. 
To keep the badge markup and styling maintainable across both the Service Guide card and the Service Details page, I would also create a small reusable component. 

Component name: CMP_ServiceStatusBadge 
[IfDefined context="autofill" type="content" key="serviceStatus"]
    <span class="status-badge" data-status="[Element context="autofill" type="content" key="serviceStatus"]">
        [Element context="autofill" type="content" key="serviceStatus"]
    </span>

[/IfDefined]

We can enhance the visuals using css like so, if needed:

.status-badge { display: inline-block; padding: 2px 8px; border-radius: 2px; font-size: 0.75rem; font-weight: 600; text-transform: uppercase; }
.status-badge[data-status="New"]     { background: #e6f4ea; color: #1e6e34; }
.status-badge[data-status="Updated"] { background: #e8f0fe; color: #1a56b8; }

<!-- 2. MENU_ServiceGuide card result design: -->

<div class="card">
    [IfDefined context="current" type="content" key="serviceIcon"]
        <img src="[Element context="current" type="content" key="serviceIcon"]" alt="" loading="lazy" />
    [/IfDefined]
    <h3>[Element context="current" type="content" key="serviceTitle"]</h3>
    [Component name="CMP_ServiceStatusBadge"]
    <p>[Element context="current" type="content" key="serviceSummary"]</p>
    <a href="[URLCmpnt context="current" type="content"]"> Read more </a>
</div>
this is to my understanding is the result card in the menu loop/rendering template.

<!-- 3. PT_ServiceDetails: -->

<article class="service-detail">
  <h1>
  [Element context="current" type="content" key="serviceTitle"]
  [Component name="CMP_ServiceStatusBadge"]
  </h1>

  <div class="summary">[Element context="current" type="content" key="serviceSummary"]</div>
  <div class="body">[Element context="current" type="content" key="serviceBody"]</div>

  <h2>How to use</h2>
  <div class="steps">[Element context="current" type="content" key="serviceSteps"]</div>

  <a class="cta" href="[Element context="current" type="content" key="serviceLink"]">Start service</a>
</article>
```

_Rationale:_
Strictly speaking, a status could be dynamically driven using data- attributes and the publishedAt or updatedAt fields. However, considering this is a product change that specifically wants an optional field, I chose the solution above.

I would only choose the date-driven approach if Product defines an exact rule, such as “show New for 30 days after publish” or “show Updated for 14 days after a meaningful update.” Without that rule, modified dates can be noisy because minor edits, metadata changes, or republishing could incorrectly show a service as updated.

For this requirement, I believe the current solution adheres more closely to the product-scoped design, as I am assuming the content team will be responsible for deciding which services should have a status badge. I wanted to state the alternative to show my thought process while approaching this change request, but I would keep the badge server-rendered through WCM rather than making JavaScript the source of truth.

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
To enable the `featured` capability, I would add a `isFeatured` property which is a boolean toggle.

For ordering within the strip, I would add another optional field to AT_Service:
- `featuredOrder` (Number): controls the order of featured services inside the strip.

Note: The requirement says featured services should appear in their "natural priority order", but the current `AT_Service` model does not define a priority field. The current Service Guide is sorted by title, but title sorting is not the same as priority sorting. Therefore, I would make that priority explicit with `featuredOrder` and sort the featured menu by `featuredOrder` ascending, with `serviceTitle` as a fallback tie-breaker.

I would introduce no further changes to the Authoring Templates. This is the min. change required to enable the new featured capability.
The existing Service Categories taxonomy can continue to drive the category grouping: 
 - Individuals
 - Business
 - Vehicles
 - Travel
 - Documents

Component approach:
I would use a combination approach. That is, I would extract the service card into a reusable component (CMP_ServiceCard). Then, for each category, define a menu. Each menu's result design references CMP_ServiceCard via a [Component] tag. 
In detail, I would use several WCM menus and a shared service card component. I would create
 - MENU_FeaturedServices
 - MENU_ServiceCategory_Individuals
 - MENU_ServiceCategory_Business
 - MENU_ServiceCategory_Vehicles
 - MENU_ServiceCategory_Travel
 - MENU_ServiceCategory_Documents

All of these menus would reuse the same card component:
CMP_ServiceCard: 
<div class="card">
    [IfDefined context="current" type="content" key="serviceIcon"]
        <img src="[Element context="current" type="content" key="serviceIcon"]" alt="" loading="lazy" />
    [/IfDefined]
    <h3>[Element context="current" type="content" key="serviceTitle"]</h3>
    [Component name="CMP_ServiceStatusBadge" ]
    <p>[Element context="current" type="content" key="serviceSummary"]</p>
    <a href="[URLCmpnt context="current" type="content"]">Read more</a>
</div>

The MENUS are here:
<!-- MENU_FeaturedServices, 
    type=AT_Service, scope=Services site area, filter=isFeatured = true, sort: featuredOrder ASC,serviceTitle ASC as fallback
<!-- Header -->
<section class="featured-strip">
    <h2>Featured services</h2>
    <div class="card-grid">
    <!-- Result Design -->
    [Component name="CMP_ServiceCard"]
    <!-- Footer -->
    </div>
</section>


<!-- MENU_{CATEGORY} where CATEGORY is one of: Individuals, Business, Vehicles, Travel, Documents 
 -type=AT_Service, scope=Services site area, filter=Service Categories contains {CATEGORY} Sort: - serviceTitle ASC -->

<!-- Header -->
<section class="service-section">
    <h2>{CATEGORY}</h2>
    <div class="card-grid">
    <!-- Result Design -->
    [Component name="CMP_ServiceCard" ]
    <!-- Footer -->
    </div>
</section>
Each category-scoped menu follows this same pattern, with {CATEGORY} replaced by Individuals, Business, Vehicles, Travel, or Documents.
Regarding caching, since this is a high-traffic landing page and services change infrequently, I would use shared/site-level caching rather than session-level caching.

Using several category-scoped menus gives us more granular cache invalidation. If a service changes, only the affected category menu and possibly the featured menu need to be invalidated.

The trade-off is that the first uncached request may run several menu queries, one for Featured and one for each category, sort of like a 'cold-start' problem. Given the category list is small and fixed, I think that is an acceptable trade-off for simpler queries, reusable rendering, and better cache control.
```

_Rationale:_
While this solution may look less dynamic than a single grouped menu, it keeps the responsibilities clear. The part that changes per section is the query. The service card rendering itself stays the same through `CMP_ServiceCard`. That means the WCM components stay easy to reason about, and we avoid adding client-side grouping logic or using the REST API unnecessarily.

I would only move to a more dynamic approach if categories were frequently changing or managed by non-developers in a way that required automatic section creation.

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

*Issues found (with impact):*

1. The menu scope is `ALL libraries`.
   Impact: this can query unrelated content, increase load, and potentially expose content outside the intended Services area.
   Fix: scope the menu to the Services site area/library only.

2. The menu has `max results=0`, meaning unlimited results.
   Impact: this can become slow as the number of services grows, especially on a high-traffic page.
   Fix: use a sensible max result count, pagination, or a grouped layout with controlled section queries.

3. The menu has `caching=none`.
   Impact: every request triggers fresh WCM queries, which is expensive for a high-traffic landing page.
   Fix: use shared/site-level caching and invalidate the relevant cache entries on publish/update.

4. The menu has `sort=none`.
   Impact: the result order is not predictable and may change between requests.
   Fix: sort by `serviceTitle ASC`, or by an explicit priority field if Product defines one.

5. The outer layout uses inline styles.
   Impact: this bypasses the design system and makes the layout harder to maintain.
   Fix: use CSS classes and shared styles instead.

6. The card uses inline styles.
   Impact: card styling becomes duplicated and difficult to update consistently.
   Fix: use the existing `card` class and keep styling in the stylesheet/style component.

7. The image URL is hardcoded to an internal CMS URL.
   Impact: every card gets the same image, the URL may break outside the internal environment, and it exposes internal infrastructure details.
   Fix: render the service’s own `serviceIcon` element.

8. The image is not conditional.
   Impact: services with no icon may render broken images.
   Fix: wrap the image in an `IfDefined` check.

9. The image has no `alt` attribute.
   Impact: this is an accessibility issue.
   Fix: add an `alt` attribute. If the icon is decorative, use `alt=""`.

10. The title uses `[Property context="current" field="title"]`.
    Impact: this may render the WCM system title instead of the authored service title.
    Fix: use the `serviceTitle` element from `AT_Service`.

11. The link is hardcoded to `/wps/myportal/details`.
    Impact: every service card links to the same URL instead of linking to the current service detail page.
    Fix: generate the URL for the current service content item.

12. The result design does not include the status badge.
    Impact: it does not meet Change Request 1.
    Fix: include `CMP_ServiceStatusBadge`.

13. The context handling needs to be consistent.
    Impact: if nested reusable components use the wrong context, they may render the wrong content item.
    Fix: in the menu result design, treat `current` as the current service item and pass that same context into reusable components.

_Corrected markup:_

```
*Corrected markup:*


[Component name="ServiceCardStyles"]
<!--
  MENU_ServiceGuide config:
  - type=AT_Service
  - scope=Services site area
  - max results=controlled by business requirement or pagination
  - caching=shared/site-level cache
  - sort=serviceTitle ASC
-->
[Component name="MENU_ServiceGuide"]

<!-- MENU_ServiceGuide Header Design -->

<div class="card-grid">

<!-- MENU_ServiceGuide Result Design -->

<div class="card">
  [IfDefined context="current" type="content" key="serviceIcon"]
    <img
      src="[Element context="current" type="content" key="serviceIcon"]"
      alt=""
      loading="lazy"
    />
  [/IfDefined]

  <h3>
    [Element context="current" type="content" key="serviceTitle"]
  </h3>
  [Component name="CMP_ServiceStatusBadge"]
  <p>
    [Element context="current" type="content" key="serviceSummary"]
  </p>
  <a href="[URLCmpnt context="current" type="content"]">
    Read more
  </a>
</div>

<!-- MENU_ServiceGuide Footer Design -->

</div>
```

---

## Trade-off Questions

1. For the category-grouped layout in CR2, compare doing it with **one menu vs. several category-scoped menus vs. the WCM REST API + custom rendering**. What drives your choice, and what does each cost you?
2. The Service Guide is high-traffic; services change infrequently but a newly added service must appear **quickly**. Describe your caching and/or pre-render strategy and how you'd invalidate it on publish.

### ➡️ Your answer



_1._
One menu: This is the simplest option from a component-count perspective. It gives us one query and one cache object.
However, grouping by taxonomy can become awkward inside a single result design. We would need the menu to either support grouped output directly or add custom logic to detect category changes and print section headings.
It also makes the Featured services strip less clean, because Featured is a different presentation rule from the normal category grouping.
I would only choose this approach if WCM provides reliable built-in grouped rendering for this use case.

Several category-scoped menus: This is the approach I would choose for this exercise. I would have one featured menu and one menu per category. Each menu has a simple query, and all menus reuse the same `CMP_ServiceCard` component.
The trade-off is that we create multiple WCM menu components. Adding a new category would mean adding another menu or making the composition more dynamic. There is also a cold-start cost because the first uncached request may run several menu queries. However, because the taxonomy is small and fixed, I think this is acceptable. This approach gives us simple queries, predictable rendering context, and granular caching.

WCM REST API: The REST API is more appropriate for a decoupled frontend, a mobile client, or a heavily interactive experience where WCM is used as a headless CMS. In this case, the site is already rendered server-side through WCM/Portal templates. Using the REST API would add extra complexity: custom data fetching, API caching, error handling, routing, SEO concerns, and duplicated rendering logic. So I would not use the REST API for this requirement unless the wider architecture was already moving toward a decoupled frontend, or potentially have an island for custom rendered contnet here.

_2._

Because the Service Guide is high-traffic and services change infrequently, I would use shared/site-level caching for the guide and menu fragments. 

A newly added service must appear quickly, so I would not rely only on a long TTL. Instead, publish/update/unpublish events should invalidate the affected cache entries.

For a changed service, I would invalidate:
* the service detail page
* the service’s category menu
* the featured menu if `isFeatured` changed
* the overall Service Guide page or CDN object if the full page is cached

I would also consider a short safety TTL and cache pre-warming after publish, so the first real user request does not pay the full rendering cost.

This gives a good balance: high cache hit rate for normal traffic, but fast visibility when content is published or updated.
