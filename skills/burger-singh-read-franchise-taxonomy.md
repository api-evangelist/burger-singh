---
name: Read the Burger Singh franchise taxonomy
description: >-
  Extract the franchise investment brackets and store formats that Burger Singh encodes in its
  category taxonomy, and understand precisely where that data stops being machine-readable.
api: openapi/burger-singh-taxonomy-api-openapi.yml
operations:
  - listCategories
  - getCategory
  - listTaxonomies
  - searchContent
  - getPage
generated: '2026-08-08'
method: generated
---

# Read the Burger Singh franchise taxonomy

Burger Singh grows through franchising, and the one place that offer is genuinely structured is the
WordPress category taxonomy. This skill gets it out — and tells you honestly how far it goes.

Base URL: `https://www.burgersinghonline.com/wp-json`. No credential required.

## Step 1 — Confirm the taxonomy registration (`listTaxonomies`)

```
GET /wp/v2/taxonomies
```

Verified live: `category` (rest_base `categories`) applies to `post`, `inthenews`, `hotdeals` and
`attachment`. `post_tag` applies to `post` and `attachment`. Read this first — it tells you which
content types the terms classify, which is the whole point of step 3.

## Step 2 — Pull every term (`listCategories`)

```
GET /wp/v2/categories?per_page=20&_fields=id,slug,name,count
```

All 13 terms come back in one request. They fall into three groups, and none of them are editorial:

**Franchise investment brackets** — the price of entry, as data:

| Term | slug | count |
|---|---|---|
| Less than 26 Lacs | `less-than-26-lacs` | 0 |
| 26 to 60 Lacs | `26-to-60-lacs` | 14 |
| 60 Lacs to 1 Crore | `60-lacs-to-1-crore` | 2 |

**Store formats** — the operating models the brand franchises:

| Term | slug | count |
|---|---|---|
| Dine-in Only | `dine-in-only` | 0 |
| Dine-in + Take Away | `dine-in-take-away` | 0 |
| Express Model | `express-model` | 0 |
| Food Court | `food-court` | 3 |
| High Street | `high-street` | 13 |

**Promotions and press** — `Hot Deal` (16), `Regular Deal` (0), `Latest news` (3), plus the
housekeeping terms `Future Post` (1) and `Uncategorized` (0).

Use `getCategory` (`GET /wp/v2/categories/{id}`) when you need a single term's `description` and
`link`.

## Step 3 — Know exactly where this stops

This is the part that matters, and it is the part an agent gets wrong.

**The counts are real; the objects behind them are not reachable.** `26 to 60 Lacs` reports
`count: 14` and `High Street` reports `count: 13`, but those objects live in the `hotdeals` and
`inthenews` custom post types, and **neither is REST-exposed**:

```
GET /wp/v2/hotdeals    -> 404 rest_no_route
GET /wp/v2/inthenews   -> 404 rest_no_route
GET /wp/v2/store_locator -> 404 rest_no_route
GET /wp/v2/medialist   -> 404 rest_no_route
```

So the taxonomy is a readable *index of an unreadable corpus*. You can say with evidence that Burger
Singh franchises at three investment tiers and five store formats. You **cannot** say from this API
which outlets run which format, what a specific franchise costs, or what the current deals are.

Do not infer a per-outlet mapping from term counts. A count is the number of objects assigned to a
term across all types the taxonomy applies to; it is not an outlet count.

**The tags collection is empty.** `GET /wp/v2/tags` returns `X-WP-Total: 0`. `getTag` has nothing to
dereference. Do not report an absent tag as a failed lookup.

## Step 4 — Fall back to the HTML pages for the narrative

For the parts the taxonomy cannot carry, the pages are the source — fetch them with
`searchContent` then `getPage`:

- `franchise` — the franchise offer.
- `hot-locations` — priority markets for new franchisees.
- `property-partners` — landlord/property-owner intake.
- `bulk-order` — bulk and catering intake.
- `co-invest-franchise-gujarati` / `-kannada` / `-marathi` — regional-language variants of the
  co-invest offer. Their existence is itself a finding: the franchise motion is being run in
  regional languages, not just English.

Treat these as HTML. `content.rendered` is theme-heavy; strip markup before reasoning over it.

## The gap worth naming

Registering `store_locator`, `hotdeals` and `inthenews` with `show_in_rest` would make the outlet
estate, the live promotions and the press record machine-readable without Burger Singh publishing a
single new document — the content already exists and already has public sitemaps. That is the
cheapest available improvement to this provider's agent surface. See
`data-model/burger-singh-data-model.yml`.

## Error contract

`{code, message, data:{status}}`, not RFC 9457. `per_page` outside 1–100 returns
`400 rest_invalid_param`; an unknown term id returns `404 rest_post_invalid_id`; a non-exposed
collection returns `404 rest_no_route`. Full catalog:
`errors/burger-singh-problem-types.yml`.
