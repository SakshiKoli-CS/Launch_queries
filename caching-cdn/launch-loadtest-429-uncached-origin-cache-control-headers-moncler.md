QUESTION

Customer **Moncler S.p.A.** (Stack `blt61bda6b71f29a239`, SF case **00053488**, JIRA **CD-9167**) ran **performance/stress tests** on `www.monclergroup.com` (hosted on Launch) and saw large error volumes they wanted explained:

- First run (backend test that **bypassed Akamai and hit the GraphQL endpoint directly**): **11,812 errors** — mostly **`SSLHandshakeException` "Remote host terminated the handshake"** (11,335) and **`azure-eu-assets.contentstack.com:443 failed to respond`** (415), concentrated on **GetInvestorPage** (1.32%) and **DownloadDocument** (1.17%).
- **Discrepancy:** customer's error rates were **far higher than what Contentstack logs showed** (Launch saw `200`s on the investor page, only a few unrelated `404`s, no asset failures).
- Later runs (2K / 8K concurrent users): site **became inaccessible**, and in the **development environment (custom logging)** they observed **every request as a cache MISS** (`X-Cache` and Cloudflare level) → **all requests reached origin → `429 Too Many Requests`**. They asked: **is there a visit rate limit? Is the 429 because all traffic comes from one IP? Could the `date` filter be defeating caching?**

_Keywords: load test errors, stress test, SSLHandshakeException remote host terminated handshake, azure-eu-assets failed to respond, 429 Too Many Requests, cache MISS X-Cache Cloudflare, all requests reach origin, GraphQL endpoint rate limit, asset CDA rate limit, cache-control headers dynamic content, date filter cache key, inform Launch before load testing, no rate limit on visits, 503 cached briefly._

ANSWER

## Inform Launch before any load/performance test

- **Performance and load testing must be coordinated with the Launch team beforehand** — see the [load testing doc](https://www.contentstack.com/docs/developers/launch/load-testing). Several runs here happened without prior notice.

## Root cause — pages weren't cached → traffic hit origin → CDA/asset rate limits returned 429

- **No rate limit on website visits.** Launch can serve **far more visitors** than the load test generated — visitor volume itself isn't the problem.
- The real issue: **the application's pages were not cached**, so **every request reached the application origin**, which then **called the CMS/assets (CDA)** — and **CDA/asset endpoints have rate limits**, returning **`429 Too Many Requests`** once exceeded.
- **If caching were in place on the application, requests would never reach the CMS** in that volume. **Fix is in the customer's control: add `Cache-Control` headers** on the application's pages (dynamic-content caching). See the [Launch caching guide — dynamic content with cache headers](https://www.contentstack.com/docs/developers/launch/caching-guide-for-contentstack-launch#dynamic-content-with-cache-headers).

## "Everything is a cache MISS" was factually incorrect — caching does work

- Edge stats from the 1-hour stress window: **`200` = 1,188,713 (1,103,915 HIT / 84,798 MISS)**, **`429` = 744,064 (all MISS)**, **`503` = 2,840**, **`504` = 159**, **`502` = 15**. Peak **CDN ~5,984 RPS**; origin **GraphQL peaked ~234 RPS**; the **auth layer returned 429 for requests over the rate-limit quota.**
- So **a large share of requests were served by the CDN (HIT)** — the claim that nothing was cached is wrong.
- **Caching condition:** a request is cached **only when the query AND its variables are exactly the same**. The **429s depend on how requests are parallelized** — e.g. **300 unique requests fired simultaneously WILL cause 429s**, but if 200 of them are already cached, only ~100 reach origin. The **`date` filter (and other query variables) change the cache key**, so varying them produces fresh MISSes that hit origin.
- **`5xx` responses are cached for ~5–10s** to let services recover — so HITs against cached 5xx are expected.

## The SSL/handshake + "failed to respond" errors (first run)

- The first run **bypassed Akamai and hit the GraphQL endpoint directly** (a backend stress test). The **`SSLHandshakeException` / `:443 failed to respond`** errors didn't correspond to Launch-side logs (Launch saw `200`s, no asset failures) → these are **client/CDN-path-side** under extreme direct load, not Launch origin failures. Requested **total request count + Akamai logs** to correlate.
- For the later **503s during the 8K test**, Launch saw **no corresponding 503s / origin failures** in its logs → recommended **reviewing Akamai CDN logs** for the source of those 503s.

## Generalized guidance (for future similar queries)

- **Load test shows mass `429`s and "everything is a cache MISS"** → the **app pages aren't cached**, so traffic hits origin and trips **CDA/asset rate limits**. **There is no Launch visitor rate limit** — the cap is at the **CMS/CDA** the origin calls. **Fix: add `Cache-Control` headers** on the app pages so the CDN serves them and requests never reach CDA at that volume.
- **A request is cached only if query + all variables match exactly.** **Query variables like a `date` filter are part of the cache key** — varying them = fresh MISS = origin hit. High **parallelism of unique requests** guarantees some 429s regardless.
- **Don't trust "0% cached" claims at face value** — pull **edge HIT/MISS stats**; here >1.1M of 1.18M `200`s were HITs.
- **Errors that don't appear in Launch logs** (SSLHandshake, direct-endpoint failures, CDN 503s) point at the **client/CDN path** (e.g. test bypassing Akamai, or Akamai-side) — **correlate with Akamai/CDN logs**, not just Launch.
- **Always coordinate load tests with Launch first.**
- Related: [load-test response-time spike (buffered responses)](../logs-monitoring/launch-load-test-response-time-spike-buffered-responses.md), [cache priming](launch-cache-priming-failures-rate-limits-batching.md), [CF1001 size limits incl. CDA 50MB](../networking-connectivity/launch-cf1001-request-size-error-header-limit-9kb-vs-cda-response-50mb.md).

## Status

- **Resolved / explained.** Cause of the 429s: **uncached app pages → origin → CDA/asset rate limits**, **not** a Launch visitor limit. Fix (customer-side): **add `Cache-Control` headers** for dynamic-content caching; remember **query+variables (e.g. `date`) define the cache key**. Edge stats showed **most requests were CDN HITs** (the "all MISS" claim was incorrect). Platform deemed **functioning as expected**; recommended **Akamai logs** for the SSLHandshake/503 errors not seen on the Launch side. Customer gave **positive feedback** after a successful corporate event launch. Tracked **CD-9167**.
