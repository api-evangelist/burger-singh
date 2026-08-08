---
name: Find and read Burger Singh brand content
description: >-
  Answer a question about Burger Singh — the menu, the franchise offer, bulk orders, an outlet, the
  terms — by searching its site content, fetching the winning page, and citing it with the site's own
  SEO / schema.org metadata.
api: openapi/burger-singh-search-api-openapi.yml
operations:
  - searchContent
  - getPage
  - listPages
  - getSeoHead
generated: '2026-08-08'
method: generated
---

# Find and read Burger Singh brand content

The shortest path from a natural-language question ("what's on the Burger Singh menu?", "how much
does a Burger Singh franchise cost?", "how do I place a bulk order?") to a citable URL with real
content. Base URL: `https://www.burgersinghonline.com/wp-json`. No credential required.

**Domain warning first.** `burgersingh.com` is a parked domain that answers HTTP 200 for every path,
including paths that do not exist. If you fetch it you will get a lander stub and believe you found
something. The company site is `burgersinghonline.com`.

## Step 1 — Search (`searchContent`)

```
GET /wp/v2/search?search=franchise&per_page=10&subtype[]=page
```

Returns lightweight records only — `id`, `title`, `url`, `type`, `subtype`. That is deliberate: it
is an index, not the content. On this host every searchable object is a page (`subtype: page`);
there are no posts.

Beware the titles. Several page titles on this site are raw slugs rather than prose — a search for
"franchise" returns `co-invest-franchise-marathi`, `co-invest-franchise-kannada`,
`co-invest-franchise-gujarati` before it returns the real `Franchise` page, because those are
language variants of the same offer. Rank by `url` slug, not by title text, and prefer the shortest
slug that matches.

## Step 2 — Fetch the winner (`getPage`)

```
GET /wp/v2/pages/{id}
```

`content.rendered` is HTML, and on this site it is theme-heavy — the page bodies carry large blocks
of markup around comparatively little prose. Strip tags before reasoning over it, and do not treat
absence of a fact in `content.rendered` as evidence the fact is absent from the site: several pages
render their substance from page-builder fields rather than post content.

`excerpt.rendered` is usually empty here. `featured_media` is `0` on most pages — use the media
library (see the harvest skill) rather than expecting a hero image on the record.

## Step 3 — Browse instead of searching, when the question is categorical

If the question is "what pages exist", skip search entirely:

```
GET /wp/v2/pages?per_page=20&_fields=id,slug,link,title
```

All 20 pages come back in one request (`X-WP-Total: 20`). The estate is small enough to enumerate,
which is faster and more reliable than guessing search terms. The families are: menu
(`menu`, `burgers`, `fries-and-sides`, `desserts`, `beverages`), franchise (`franchise`,
`property-partners`, `hot-locations`, `bulk-order`, plus the three `co-invest-franchise-*`
translations), estate (`store-locator`, `stores-list`), service (`feedback`, `complaint`) and legal
(`terms-and-conditions`).

## Step 4 — Cite it (`getSeoHead`)

```
GET /yoast/v1/get_head?url=https://www.burgersinghonline.com/menu/
```

Returns `json.title`, `json.canonical`, the Open Graph fields, and `json.schema['@graph']` — the
site's own schema.org description of the page. Use `json.canonical` as the citation URL rather than
the URL you happened to fetch, and `json.title` rather than the WordPress `title.rendered`, which on
this site is sometimes a slug.

## What you cannot get, and must not fabricate

These are the questions this API **cannot** answer. Say so rather than inferring:

- **Menu items and prices as data.** The menu pages are HTML. There is no menu-item resource, no
  price field, and no ordering API on this domain.
- **The outlet estate.** `store_locator` is a registered custom post type with its own public
  sitemap, but it is not REST-exposed: `GET /wp/v2/store_locator` returns `404 rest_no_route`. Read
  the `store-locator` / `stores-list` pages as HTML, and note that the store sitemap's `lastmod` is
  2022 — the published outlet list is stale relative to the company's stated growth.
- **Current deals.** `hotdeals` is likewise not REST-exposed. The `Hot Deal` category term is
  readable (16 objects assigned) but the objects behind it are not.
- **Press coverage.** `inthenews` — same story.

## Error contract

Errors are `application/json` with `{code, message, data:{status}}` — **not** RFC 9457
problem+json. The ones you will hit:

| Status | `code` | Meaning |
|---|---|---|
| 400 | `rest_invalid_param` | `per_page` outside 1–100 is the usual cause. |
| 404 | `rest_post_invalid_id` | The page id does not exist. |
| 404 | `rest_no_route` | You asked for a collection that is not REST-exposed. |
| 401 | `rest_forbidden` | Administrative route. There is no credential you can obtain. |

Full catalog: `errors/burger-singh-problem-types.yml`. Conventions:
`conventions/burger-singh-conventions.yml`.
