QUESTION

Customer **Next** (Stack `blt9c9c23f853325239`, Azure EU, SF case **00052826**) reports **publishing entries doesn't reflect the latest changes** on the website or in **Live Preview**. The publish succeeds and a **newer version is created (e.g. v6)**, but the **environment API and Live Preview keep returning an older version (e.g. v4)**; sometimes updates appear only after a **significant delay** → **stale content served.**

- **Cloudflare is serving a cached response** for page requests across all their Launch subdomains (`staging-/production-next-international/uk.eu-azcontentstackapps.com`).
- **Direct Delivery API calls return the latest data**, and **running locally is fine** — only the **deployed Launch URLs** serve stale content.
- Blocking a go-live for two countries.

_Keywords: published entry not reflecting, stale content served, older version v4 vs v6, Live Preview stale, Cloudflare caching page responses, no Cache-Control headers, cached up to 1 year default, max-age 1y browser caching, s-maxage, CDN cache revalidation, purge cache automation, direct delivery API fine local fine only deployed stale._

ANSWER

## Cause — pages cached at the CDN because no `Cache-Control` is set

- **Expected behavior when responses are cached at the CDN layer.** Since the **app sets no explicit `Cache-Control` headers**, Cloudflare treats the page responses as **static assets** and may **cache them for up to 1 year by default** → **old versions (v4) served** even though **newer versions (v6) are published.**
- This is exactly why **direct Delivery API calls** (latest) and **local runs** (no CDN) are fine, while **only the deployed Launch URLs** serve stale content — the staleness lives in the **CDN cache**, not the CMS/CDA.
- Also noted on the **production** sites: `Cache-Control: max-age=1y`, which causes **long-term browser caching** too.

## Fix — set proper cache headers + revalidate/purge

1. **Add `Cache-Control` headers** on responses to control TTL — e.g. **`no-store`** (don't cache) or **`s-maxage`** with a **lower TTL**. Docs: Cache-Control headers.
2. **Use CDN Cache Revalidation** so updated content is re-fetched from origin on change; and/or configure an **automation to purge the Launch cache after a CMS update** so the next request bypasses the cached response and gets the latest content. Docs: Revalidate CDN Cache.
3. **Purge cache when needed** — **time-based**, **path-based**, or **tag-based** (cache tags) purging.
4. **Prefer `s-maxage` over `max-age=1y`** — `s-maxage` caches at the **CDN layer** while keeping **browser caching** under control (avoid the year-long browser cache).

## Generalized guidance (for future similar queries)

- **"Published changes don't show on the deployed site, but Delivery API/local are fine"** → **CDN is serving a stale cached page.** It's **not** a CMS/publish problem.
- **Root cause is almost always missing/over-long `Cache-Control`** — with **no header, the CDN may cache pages up to ~1 year by default**; with **`max-age=1y`** you also get a year of **browser** caching. **Set `s-maxage` (sensible TTL) or `no-store`**, and use **revalidation / purge** (path/tag/time) to push fresh content immediately.
- **`max-age` vs `s-maxage`:** `max-age` = browser cache; `s-maxage` = CDN/shared cache. For dynamic/CMS-driven pages, control the **CDN** with `s-maxage` and avoid long `max-age`.
- Related: [redirect cached 1 year (s-maxage)](launch-redirect-loop-308-cached-redirect-smaxage-1year-cache-control.md), [load-test 429 / Cache-Control](launch-loadtest-429-uncached-origin-cache-control-headers-moncler.md), [Next.js RSC payload behind CDN](launch-nextjs-app-router-rsc-payload-instead-of-html-cdn-rsc-affected-paths.md), [Revalidate CDN Cache](launch-revalidate-cdn-cache-one-parameter-per-request.md).

## Status

- **Resolved.** Cause: **page responses cached at the CDN with no `Cache-Control`** (default up to ~1 year), serving stale published versions; **Delivery API/local fine** confirmed it was CDN-layer. Fix: **add `Cache-Control` (`s-maxage`/`no-store`)** + **revalidation/purge**, and **replace `max-age=1y` with `s-maxage`**. Customer **confirmed implementing cache-control headers resolved it.**
