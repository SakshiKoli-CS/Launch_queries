QUESTION

Customer **Philips** is optimizing **CDA (Content Delivery API) usage**. They currently use an **in-memory LRU cache** (package, **100 items** — a conservative size) to cache requests and asked:
- What's their **average/peak memory consumption** today, and **can the LRU cache size be lifted**?
- (Context) They **can't use Launch CDN caching** because they send the **hostname in a custom header**, and **custom headers aren't valid for CDN cache keys** (see [cache-key custom-header entry](launch-cache-key-custom-header-locale-domain-queryparam-workaround.md)).

_Keywords: CDA usage optimization, reduce CDA calls, in-memory LRU cache, cache not shared across instances, short-lived instances, peak memory 800MB 1GB, proxy CDN caching layer, centralized cache, cache wrapper, repeated CDA calls._

ANSWER

## Memory today — ~800 MB to 1 GB peak per instance

- **Peak memory per instance is ~800 MB–1 GB**, with several instances operating **close to the upper threshold**. (Per-instance metrics — `startTime`, `endTime`, `peakMem`, `invocations` — reflect individual instance lifecycles.)

## Why the in-memory LRU cache is largely ineffective (the real issue)

- **Launch spawns multiple short-lived instances** for scalability; **each operates independently** and makes its own CDA calls.
- Each **new instance starts with an empty cache**, and **cache is NOT shared across instances**. So an **in-memory LRU cache is scoped to a single instance lifecycle** and doesn't persist across newly created instances → **repeated CDA calls** across instances.
- **Therefore, increasing the LRU size (100 → more) won't meaningfully help** — the limitation is the **per-instance, non-shared, short-lived** nature of the cache, not the item count.

## Recommendation — a dedicated proxy layer with CDN-based caching

Centralize caching **outside** the app instances so all instances (including new ones) share it:

```
User
  ↓
Primary CDN (Cloudflare)
  ↓
Application Instance
  ↓
New Launch Environment (proxy: forwards to CMS, returns cacheable responses with Cache-Control set)
   ├── Cache Hit  → response returned
   └── Cache Miss → CDA/CMS → cache at proxy CDN → response returned
  ↓
User
```

- Requests flow through a **proxy layer**: serve **cached responses** when available; on a **miss**, fetch from CDA/CMS, **cache it**, and return — so **every instance benefits from previously cached responses**, sharply cutting CDA calls.
- Reference implementation (caching wrapper pattern around API calls): `github.com/contentstack-launch-examples/management-api-cache-wrapper`.

## Generalized guidance (for future similar queries)

- **"Can we increase our in-memory/LRU cache to reduce CDA calls?"** → On Launch, **in-memory cache doesn't persist or share across the short-lived, multiple instances**, so bumping its size has **limited effect**. The fix is a **shared/centralized cache**, not a bigger per-instance one.
- **Reduce CDA usage durably** with a **proxy layer + CDN caching** (a Launch environment that proxies to CMS and returns `Cache-Control`-tagged responses), so cache is shared across all instances.
- If **Launch CDN caching can't be used** (e.g. cache-key needs a custom header) → the proxy-layer caching pattern is the workaround; see the [custom-header cache-key entry](launch-cache-key-custom-header-locale-domain-queryparam-workaround.md).
- Peak per-instance memory of **~800 MB–1 GB near the limit** is itself a signal to move caching **off the instance**.

## Status

- Shared peak memory (~800 MB–1 GB/instance) and explained the in-memory LRU limitation (per-instance, non-shared, short-lived). Recommended a **proxy + CDN caching layer** (with the example repo) to centralize caching and reduce CDA calls.
