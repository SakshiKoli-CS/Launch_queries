QUESTION

Customer **Conforama** (Org `bltd55bb837b010f99f`, Project `679389cd4bff8d75fbab3157`, EU, SF case **00049933**) reports a **critical production issue: user-specific session data is being mixed between users.** The session route **`/api/v1/corecommerce/guest`** returns **inconsistent responses** — `store_name` and other user data **sometimes belong to a DIFFERENT user** than the one making the request. **Exposes user data to other users.** Session cookies: `JSESSIONID`, `ROUTE`. The response had **`cache-control: no-cache`** and **`cf-cache-status: EXPIRED`**.

_Keywords: user data leaked between sessions, response belongs to another user, session route inconsistent, /api/v1/corecommerce/guest, cache-control no-cache insufficient, no-store fix, cf-cache-status EXPIRED, CDN caching personalized response, no-cache allows caching with revalidation, no-store no caching, in-memory cache React.cache, Delivery SDK cache policy, inconsistent across deployments/routes._

ANSWER

## Root cause — `Cache-Control: no-cache` was insufficient; the CDN cached/reused personalized responses

- The responses were set to **`no-cache`**, but **`no-cache` does NOT mean "don't cache."** It means **cache-with-revalidation** — the CDN may **store and reuse** the response under certain conditions. So **personalized/session responses got cached and served to other users** → the data leak.
- **Fix: `Cache-Control: no-store`** — explicitly tells the CDN **not to cache the response at all.** After applying `no-store` to the affected API routes, the issue stopped.
- **`no-cache` vs `no-store`:** `no-cache` = **caching allowed, must revalidate**; `no-store` = **never cache.** For **per-user / session / personalized** responses, use **`no-store`.**

## Why the confusing signals

- **Only `www.conforama.fr` (+ the default Launch domain) leaked, not a fresh test project** (`…-b3d2dd`) → and the leaking domains had the `JSESSIONID`/`ROUTE` cookies while the test one didn't. It looked app-specific/environment-specific, but the real variable was **which responses were cacheable + being cached** at the CDN.
- **Inconsistent across deployments and across routes** → because caching of a personalized response is opportunistic; different deploys/routes cached different user payloads. This inconsistency is a hallmark of **a caching problem, not deterministic app logic.**

## Ruling in/out other layers (investigation trail)

- Launch **CDN reports showed no caching at the Launch CDN layer** for the route → pushed the investigation to **app-layer / client caching**.
- App had **`React.cache` in the Node/SSR layer** (removed) and a **`Response.clone: Body has already been consumed`** error (undici) — these were **investigated but not the root cause**. Also checked: **in-memory caching should be avoided at the app layer**; **Delivery SDK cache policy** can cache SSR responses (remove it if set).
- Ultimately the durable fix was the **`no-store` Cache-Control** on the personalized routes.

## Generalized guidance (for future similar queries)

- **User/session data leaking between users, inconsistent personalized responses** → a **caching problem**: a **personalized response is being cached and reused.** **Check the `Cache-Control` header** — **`no-cache` is NOT enough** (it permits cache-with-revalidation). **Use `no-store` on all per-user/session/personalized routes.**
- **`no-cache` = cache + revalidate; `no-store` = never cache.** Personalized/auth'd/session endpoints must be **`no-store`.**
- **Inconsistent-across-deploys/routes + only on the live domain** is a strong caching tell (not deterministic app logic). Also avoid **in-memory app-layer caches** (short-lived, non-shared instances) and **Delivery SDK cache policies** on SSR responses.
- Related: [stale published content / no Cache-Control](launch-stale-published-content-cdn-cache-no-cache-control-headers-next.md), [in-memory LRU ineffective → CDN/proxy](launch-cda-usage-lru-cache-ineffective-proxy-cdn-caching.md), [cache persists past s-maxage (non-standard value)](launch-cache-persists-past-s-maxage-nonstandard-format.md).

## Status

- **Resolved.** **Critical data leak** (session responses served to other users) was caused by **`Cache-Control: no-cache` being insufficient** — the CDN **cached/reused personalized responses.** Fixed with **`Cache-Control: no-store`** on the affected API routes. App-layer suspects (`React.cache`, undici `Response.clone`, Delivery SDK cache policy) were investigated but the **header (`no-store`) was the fix.** Confirmed the **`no-cache` (revalidate-allowed) vs `no-store` (never cache)** distinction to the customer.
