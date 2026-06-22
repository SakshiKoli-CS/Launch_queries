QUESTION

SI **Silvertech Inc** (implementing for customer **Fulton Bank**) asked how Launch CDN caching interacts with **server-rendered HTML that varies by cookie / personalization / audience state**, while implementing HTML caching for the Fulton site.

Understanding stated: Launch caches HTML by URL when responses include `Cache-Control: public, s-maxage=...`. Specific questions:

1. Could an **anonymous/default** cached page be served to a **personalized** user before the request reaches the Next.js app?
2. Could a **personalized** version be cached and served to **anonymous/default** users?
3. If Next.js returns `Cache-Control: no-store` when personalization cookies are present, does Launch **still check the CDN cache first**, or do those requests always reach the app?
4. Does Launch support **varying the HTML cache key by cookie, header, or query param**?
5. What is the **recommended pattern** for pages with server-rendered personalized fragments — bypass HTML caching, use client-side/API fragments, or configure cache variation?

_Keywords: Launch CDN caching, personalization, cookie-based content, cache key variation, Vary header, s-maxage public, no-store, SSR personalized HTML, anonymous vs personalized cache bleed, audience state, revalidateTag revalidatePath not supported, Revalidate CDN Cache API, cache tags, edge caching personalization._

ANSWER

## Direct answers

**1. Can cached anonymous/default HTML be served to a personalized user? → YES (risk).**
If the HTML response is cached at the CDN with cacheable headers (`Cache-Control: public, s-maxage=...`), Launch may serve the cached response for that URL **without evaluating cookies or personalization state**. A default/anonymous version can be served to a personalized user.

**2. Can personalized HTML be cached and served to anonymous users? → YES (risk).**
If personalized SSR HTML is cached under a shared URL, it can be incorrectly served to other users. **Personalized HTML should not be cached at the CDN level.**

**3. Does `Cache-Control: no-store` bypass the cache? → YES.**
`no-store` ensures the response **bypasses CDN caching** and is always served directly from origin (the Next.js app). Those requests reach the app every time.

**4. Does Launch support cache variation by cookie / header / query param? → NO.**
Launch **does not** support HTML cache variation based on cookies, headers, or session state. Caching is controlled primarily by **URL + Cache-Control headers**. (No `Vary`-by-cookie equivalent for HTML.)

**5. Recommended pattern for personalized SSR pages:**
- Use **dynamic SSR with `Cache-Control: no-store`** for any personalized or cookie-dependent page → HTML is rendered per request, eliminating any risk of cached personalized content bleeding across users.
- Apply **CDN caching only to shared data** (API/CMS layer), so reusable content is still served from the Launch CDN while user-specific rendering stays fully dynamic.

## Why this is the model (important platform limitation)

**Launch does not support Next.js-style data-cache semantics** (`revalidateTag()` / `revalidatePath()` at the application layer). Caching is handled at the **CDN layer** via standard HTTP mechanisms: `Cache-Control` headers, **Cache Tags**, CDN cache behavior, and **CDN revalidation APIs**.

Because the CDN cache key is URL-based and **cannot vary by cookie**, the only safe design is **strict separation** between "what's personalized" (never CDN-cached) and "what's shared" (CDN-cached).

## Recommended architecture

```
                     Browser
                        │
                        ▼
                  Dynamic Page (SSR, per-request)
                        │
        ┌───────────────┴───────────────┐
        ▼                               ▼
 User-specific Data              Shared Content
   (No Cache: no-store)           (CDN Cache)
        │                               │
        ▼                               ▼
   Database / User APIs          CMS Content / Product
   Auth Services                 Catalog / External APIs
                                        │
                                        ▼
                                Launch CDN Cache

   Content Updates ─▶ Launch CDN Revalidation APIs
        ├─ Cache Tag revalidation
        ├─ Path revalidation
        └─ Hostname revalidation
```

**How it works:**
- SSR pages are always rendered **dynamically per request**.
- **User-specific data is never CDN-cached.**
- **Shared CMS/API data** is CDN-cached; static assets (JS/CSS/images/fonts) are also CDN-cached.
- **Content updates trigger Launch CDN Revalidation APIs** (Cache Tag / Path / Hostname) to invalidate and refresh cached shared content.

**Cache invalidation:** use the **Revalidate CDN Cache API** (Cache Tags, path revalidation, hostname revalidation).

## Generalized guidance (for future similar queries)

- **"Will Launch serve a cached page to the wrong user (personalized↔anonymous)?"** → Yes, if the personalized HTML is CDN-cacheable. The CDN matches on **URL only** and ignores cookies/personalization. Don't CDN-cache personalized HTML.
- **"Can I vary the HTML cache by cookie/header/query?"** → **No.** No cookie/header-based cache variation for HTML on Launch. Control caching via URL + `Cache-Control`.
- **"How do I keep personalization correct but still get CDN performance?"** → `no-store` on personalized SSR pages; cache only **shared** data/content at the CDN; fetch user-specific data per request.
- **"Does `revalidateTag()`/`revalidatePath()` work on Launch?"** → **No** Next.js app-layer data-cache semantics. Use CDN-layer **Cache Tags + Revalidate CDN Cache API** instead.
- **`no-store` guarantees origin** — requests with it always reach the Next.js app and are never served from CDN cache.

## Status

- Answers shared with the SI/customer; no further follow-up at last update. Offer of a call extended if needed.
