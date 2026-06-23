QUESTION

A customer is migrating from **Umbraco to Contentstack in phases** — run **some pages on a new Contentstack/Launch frontend** while keeping the rest on the **existing Umbraco** site for now. Constraint: they **cannot modify the existing Umbraco frontend** to pull Contentstack content.

Questions:
1. Can **Launch (or Cloudflare) rewrite certain pages** to use Contentstack data while everything else stays on Umbraco — i.e. can Launch handle this with **rewrites**?
2. What's the **recommended setup**?
3. What should be gathered from the customer to **scope** it (domain/URL structure, current hosting)? Do they need **two domains** (e.g. `protolabs.com` via Launch, `legacy.protolabs.com` via Umbraco)?

_Keywords: phased migration, incremental migration, Umbraco to Contentstack, split traffic, rewrite some paths to Launch, proxy legacy site, single domain migration, launch.json rewrites, Edge URL Rewrites, Edge Functions, gradual cutover, section-by-section migration._

ANSWER

## Yes — this is a common phased-migration pattern on Launch

Feasible using **Launch rewrites/redirects via `launch.json` or Edge Functions**, depending on routing complexity. It lets customers migrate **section-by-section / page-by-page** off Umbraco onto a Launch+Contentstack frontend **without a full cutover** and **without modifying the existing Umbraco frontend**.

## Recommended setup

- Host the **new application on Launch**, connected to **Contentstack**.
- Route the **primary domain through Launch** (this is the key requirement — see below).
- Configure **rewrite rules** so specific paths are served by the new Launch app while everything else continues to resolve to the **existing Umbraco** application.

**Example:**
```
/blog/*        → Launch + Contentstack (new app)
all other paths → existing Umbraco site
```

This keeps a **single public domain** for end users while traffic is split behind the scenes.

## Critical requirement — primary domain must point at Launch

- The **primary traffic/domain must be routed through Launch**, because **Launch is what evaluates the rewrite rules** and forwards non-migrated requests to Umbraco. If the domain doesn't terminate at Launch, Launch can't make the routing decision.
- So it's **not** strictly "two public domains" — the model is **one public domain on Launch** that proxies/rewrites unmatched paths to the Umbraco origin (the Umbraco origin can sit on a separate internal hostname). A two-domain split (`protolabs.com` vs `legacy.protolabs.com`) is a different, coarser approach and not required for path-level rewrites.

## What to find out from the customer (scoping checklist)

- **Domain/URL structure:** what's the primary public domain, and which **paths/sections** migrate first vs stay on Umbraco? (Clean path boundaries like `/blog/*` make rewrites simpler.)
- **Current hosting / DNS:** where is the domain's DNS managed, and can the **primary domain be pointed at Launch**? Is anything (CDN/WAF) currently in front of Umbraco?
- **Umbraco origin:** what **hostname/endpoint** should unmatched traffic be forwarded to (the legacy origin)?
- **Routing complexity:** simple path prefixes (→ `launch.json` rewrites) vs conditional/dynamic logic (→ Edge Functions)?
- **SEO/redirects:** any URL changes needing redirects vs transparent rewrites.

## Tooling choice

- **`launch.json` (Edge URL Rewrites/Redirects)** — for straightforward path-based rules.
- **Edge Functions** — when routing needs conditional/dynamic logic beyond static path rules.
- Refs: `launch.json`, Edge URL Redirects, Edge URL Rewrites docs.

## Generalized guidance (for future similar queries)

- **"Can Launch serve some pages from Contentstack and proxy the rest to our legacy CMS?"** → Yes — **point the primary domain at Launch** and use **`launch.json` rewrites or Edge Functions** to serve migrated paths from the Launch app and forward the rest to the legacy origin. Single public domain; no legacy-frontend changes.
- **Domain model:** the public domain terminates at **Launch** (the rewrite evaluator); the legacy origin is just a forward target — you don't need two *public* domains for path-level migration.
- **Scope first:** path boundaries, who controls DNS / can the domain move to Launch, the legacy origin endpoint, and whether routing is static (launch.json) or dynamic (Edge Functions).
