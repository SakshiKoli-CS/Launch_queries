QUESTION

Customer **Philips** has a **single Next.js app serving all domains** of their site. **Akamai** fronts all those domains and forwards requests to the Launch app with the **domain name in a custom HTTP header**, which the app uses to pick a **locale** for querying the Contentstack Delivery API.

Problem: **Launch cache keys can't include a custom HTTP header**, so requests that differ only by that locale/domain header would **share a cache entry** — meaning they can't safely use Launch caching (and **`stale-while-revalidate`**) to take pressure off the CMS CDN. Is the **no-custom-header-in-cache-key** limitation **permanent**, or possible later?

_Keywords: cache key custom header, vary cache by header, locale header cache, Akamai custom header, stale-while-revalidate, per-locale caching, can't cache by header, domain per locale, locale query param, CMS CDN pressure._

ANSWER

## The limitation

- **Launch cache keys do not support varying entries by arbitrary custom request headers.** Requests differentiated **only** by a custom locale/domain header would **share the same cache entry**, which makes `stale-while-revalidate` unsafe in that setup. (Same root limitation as [no cookie/header cache-key variation](launch-cdn-caching-personalization-no-cache-variation.md).)
- Whether **custom-header-based cache keys** will be supported in future is **to be discussed with product/engineering** — no commitment today.

## Workaround 1 — separate Launch domain per locale (recommended)

Make the **hostname** the cache differentiator instead of a header:
1. Configure **separate Launch custom domains per locale**, e.g. `philips-en.contentstackapps.com`, `philips-fr.contentstackapps.com`, `philips-de.contentstackapps.com`.
2. From **Akamai**, route each public domain to the matching locale Launch domain (`www.philips.fr → philips-fr…`, `www.philips.de → philips-de…`).
3. In Next.js, **resolve the locale from the incoming host/domain** and query the Delivery API with it.

Result: **hostname is naturally part of the cache identity**, so cache is **isolated per locale automatically** and **`stale-while-revalidate` / CDN caching work** without needing header-based cache keys.

## Workaround 2 — append locale as a query parameter

1. **Akamai rewrites** the request to append the locale, e.g. `/page → /page?locale=nl-nl`, before forwarding to Launch.
2. Next.js reads the **`locale` query param** and uses it for the Delivery API.

Result: the **URL is unique per locale**, so Launch maintains **separate cache entries** automatically → CDN caching + `stale-while-revalidate` function correctly.

## Generalized guidance (for future similar queries)

- **"Can Launch vary the cache key by a custom header (locale/domain/etc.)?"** → **No** — cache keys don't support arbitrary custom headers, so header-only-differentiated requests collide in cache. (Future support = open with product, not committed.)
- **To get per-variant caching + `stale-while-revalidate`,** make the differentiator part of the **cache identity**: either a **distinct hostname/domain per variant** (preferred) or a **unique URL/query param** per variant — both inject the distinction into the URL/host the cache keys on.
- Applies to any "fronted by Akamai/Cloudflare/external CDN passing a header" locale/multi-domain setup on a single Launch app.

## Status

- Explained the limitation and provided the **per-locale domain** and **locale query-param** workarounds; custom-header cache-key support raised as a possible **future** item with product. Customer found this helpful.
