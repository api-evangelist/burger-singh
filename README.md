# Burger Singh

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Burger Singh is an Indian quick-service restaurant chain founded in 2014 by Kabir Jeet Singh with
co-founders Nitin Rana and Rahul Seth, operated by Tipping Mr Pink Private Limited out of Gurugram,
Haryana. It sells Indian-flavoured "craft" burgers alongside fries and sides, desserts and beverages,
through a hybrid estate of company-owned and franchised outlets spanning high-street, food-court,
express and dine-in formats.

- Website: https://www.burgersinghonline.com/
- Secondary-market listing: https://www.hiive.com/securities/burger-singh-stock

## About the API surface in this profile

**Burger Singh publishes no developer program.** There is no API documentation, no developer portal,
no SDK, no CLI, no MCP server, no agent card, no webhooks, no `/.well-known/` document of any kind
and no `llms.txt`. That is not a criticism — it is a restaurant chain, and software is not the
product.

The one machine-readable interface it exposes is the **WordPress REST content API** behind its
corporate site at `burgersinghonline.com` (`/wp-json/` — 200 routes across 12 namespaces). The read
half of that surface is fully public: every operation described in `openapi/` was verified to return
HTTP 200 to an anonymous request carrying no credential on 2026-08-08. It is an incidental CMS
surface, not a product, and it is documented here for discovery purposes only.

Everything in `openapi/`, `overlays/`, `mcp/`, `skills/`, `llms/` and `agentic-access/` was derived
or generated by API Evangelist from that live surface. **None of it is published by Burger Singh**,
and the `mcp/` tool list is a candidate proposal, not an offering.

### Two things a reader should know

**`burgersingh.com` is not the company site.** It is a domain-parking lander that returns HTTP 200
for every path, including paths that do not exist, and serves a parking provider's `robots.txt` and
`llms.txt`. Those are deliberately not credited to the company. The live corporate site is
`burgersinghonline.com`.

**The business-meaningful content is the content that is *not* machine-readable.** The outlet estate
(`store_locator`), promotions (`hotdeals`), press coverage (`inthenews`) and `medialist` are all
registered custom post types with public sitemaps, but none are REST-exposed — `GET
/wp/v2/store_locator` returns `404 rest_no_route`. Registering those types with `show_in_rest` would
make the estate and the offer machine-readable without publishing anything new. See
`data-model/burger-singh-data-model.yml`.
