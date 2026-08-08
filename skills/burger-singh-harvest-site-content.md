---
name: Harvest Burger Singh site content and media
description: >-
  Mirror the pages and the 576-item media library behind burgersinghonline.com, driven by the
  discovery routes rather than a hardcoded content model, respecting the pagination, projection and
  error contract.
api: openapi/burger-singh-discovery-api-openapi.yml
operations:
  - getRootIndex
  - listTypes
  - listTaxonomies
  - listStatuses
  - listPages
  - getPage
  - listMedia
  - getMediaItem
  - listUsers
generated: '2026-08-08'
method: generated
---

# Harvest Burger Singh site content and media

A complete, polite mirror of everything anonymously readable on
`https://www.burgersinghonline.com/wp-json`. No credential required. Drive it from discovery so the
harvest keeps working when the site changes.

## Step 0 — Discover, do not assume (`getRootIndex`)

```
GET /wp-json/
```

Returns the site name (`Burger Singh`), description (`Home of Desi Indian Burgers`), `gmt_offset`
(`5.5` — India, so timestamps in records are IST-local without a zone), `site_icon_url`, the
registered `namespaces`, the full `routes` table with per-route argument schemas, and the
`authentication` object.

Two things to read out of it before you fetch anything:

1. **The route table is the contract.** 200 routes across 12 namespaces at capture time. Do not
   hardcode paths — read `routes` and intersect with what you need. Plugin namespaces
   (`yoast/v1`, `contact-form-7/v1`, `post-smtp/v1`, `psd/v1`, `adsagent/v1`, `wp-abilities/v1`)
   come and go with plugin updates; `wp/v2` is the stable core.
2. **`authentication` declares `application-passwords`.** That is an administrative site credential,
   not a developer credential, and there is no self-serve path to one. Plan for anonymous only.

Then `listTypes`, `listTaxonomies` and `listStatuses` to learn the content model:

```
GET /wp/v2/types        -> post, page, attachment, nav_menu_item, wp_block, wp_template,
                           wp_template_part, wp_global_styles, wp_navigation, wp_font_family,
                           wp_font_face
GET /wp/v2/taxonomies   -> category (post, inthenews, hotdeals, attachment), post_tag
GET /wp/v2/statuses     -> publish, acf-disabled
```

**Note the trap.** `/wp/v2/taxonomies` names `inthenews` and `hotdeals` as content types, and
`/store_locator-sitemap.xml` exists — but none of those types appear in `/wp/v2/types` and none are
REST-exposed. `GET /wp/v2/store_locator` returns `404 rest_no_route`. Discovery tells you they
exist; it does not make them fetchable. Do not build a harvester that assumes every type named
anywhere is a collection.

## Step 1 — Pages (`listPages`, `getPage`)

```
GET /wp/v2/pages?per_page=100&page=1
```

`X-WP-Total: 20`, so one request is the whole set. Read `X-WP-Total` / `X-WP-TotalPages` rather than
looping until empty, and follow the RFC 8288 `Link` header's `rel="next"` if you paginate.

Every record already carries `content.rendered`, so `getPage` is only needed for a targeted refetch.
Use `_fields` to keep the payload small when you only want the index:

```
GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,modified
```

## Step 2 — Media (`listMedia`, `getMediaItem`)

```
GET /wp/v2/media?per_page=100&page=1
```

`X-WP-Total: 576` → 6 requests at `per_page=100`. This is the largest and freshest collection on the
site (newest attachment dated 2026-07-30 at capture; the newest *page* change is 2025-07-23).

Each record gives you `source_url` (the original file), `media_details` (dimensions plus every
generated size variant), `mime_type`, `filesize` and `alt_text`. Prefer a `media_details.sizes`
variant over `source_url` when you only need a thumbnail — full-size assets here run to hundreds of
kilobytes each.

`alt_text` is frequently empty on this host. Do not synthesise alt text and attribute it to the
company.

## Step 3 — Taxonomy and authors

```
GET /wp/v2/categories?per_page=100   -> 13 terms (see the franchise-taxonomy skill)
GET /wp/v2/tags?per_page=100         -> empty
GET /wp/v2/users                     -> 1 record ("burgersingh")
```

There is exactly one author, so `Page.author` and `MediaItem.author` always resolve to it. Do not
present it as a byline — it is a site account, not a person.

## Step 4 — Optional per-URL metadata (`getSeoHead`)

For each page's canonical URL, `GET /yoast/v1/get_head?url=...` returns the rendered head plus the
schema.org `@graph`. Useful if you want the site's own structured description rather than your
parse of its HTML. One request per URL — 20 requests for the whole site.

## Pacing and etiquette

- The origin is a shared Hostinger/LiteSpeed host. Serialize, do not fan out; a second or two
  between requests is plenty for a 20-page, 576-media harvest.
- **No caching headers are emitted** — no `ETag`, no `Last-Modified`, no `Cache-Control` on the JSON.
  You cannot do conditional requests. Use the `modified` field on each record for change detection,
  and poll on the order of weeks: the site's own RSS `lastBuildDate` is 2025-08-11.
- **No rate-limit headers either.** Absence of a signal is not absence of a limit. Back off on any
  5xx and do not retry aggressively.
- Honour `robots.txt` (`well-known/burger-singh-robots.txt`): `/wp-admin/` and several
  `franchise-*` staging paths are disallowed.

## Routes to skip — all gated, no obtainable credential

`/wp/v2/settings`, `/themes`, `/plugins`, `/menus`, `/block-types`, `/templates`,
`/wp/v2/pages/{id}/revisions`, `/wp-site-health/v1/*`, `/psd/v1/*` → `401`.
`/contact-form-7/v1/contact-forms` → `403 wpcf7_forbidden`.
`/wp-abilities/v1/abilities` → `401` — the WordPress Abilities registry is installed but entirely
credentialed, so no ability schemas exist to harvest.

## Error contract

`{code, message, data:{status}}` in `application/json` — not RFC 9457. `per_page` outside 1–100 is
`400 rest_invalid_param`. One plugin route (`/post-smtp/v1/get-logs`) breaks the envelope entirely
and returns `{"success":false,"data":{"error":"..."}}`; if you harvest plugin namespaces, do not
assume `code`/`message` are present. Full catalog: `errors/burger-singh-problem-types.yml`.
