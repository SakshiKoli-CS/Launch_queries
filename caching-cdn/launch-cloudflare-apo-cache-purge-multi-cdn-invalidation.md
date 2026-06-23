QUESTION

Customer asked several Cloudflare/caching questions about Launch:

1. **Cloudflare APO:** Some indication Launch supports **Cloudflare APO (Automatic Platform Optimization)** — can you confirm APO is **not** in use?
2. **Webhooks for cache purge:** Docs mention webhook support for cache purge — does that **call the Cloudflare API directly**, or must the customer receive a webhook and transform it into an API call?
3. **API token handling:** Either way, how does Launch **collect and protect the Cloudflare API tokens** needed to authorize cache-purge calls?
4. **Follow-up — multi-CDN:** The Launch revalidation API seems specific to Launch's own CDN, so it wouldn't purge Cloudflare. **How do clients handle cache invalidation in a multi-CDN setup** (Cloudflare in front of Launch)?

_Keywords: Cloudflare APO, Automatic Platform Optimization, cache purge webhook, Cloudflare API token, Launch Cache Revalidation API, Revalidate CDN Cache, multi-CDN, external CDN in front of Launch, cache invalidation propagation, Host header forwarding, Contentstack Automate._

ANSWER

## 1. Cloudflare APO — NOT supported/exposed

- Launch **does not support or expose Cloudflare APO**. Launch uses Cloudflare as part of its managed CDN/edge infrastructure, but performance optimizations are handled through **Launch-native capabilities** — **Edge Functions, Caching controls, Cache Priming** — not APO. (Ref: Launch Caching Guide.)

## 2. Cache purge / webhooks — Launch does NOT call the Cloudflare API for you

- Launch supports cache revalidation through the **Launch Cache Revalidation API** and **does not directly invoke the Cloudflare API** on the customer's behalf.
- Revalidation can be triggered **manually or via automation** — a common pattern is **Contentstack Automate** to revalidate cached content when entries are published.
- The **Revalidate CDN Cache API** supports revalidation by **specific paths, cache tags, and hostnames**. (Ref: Launch Cache Revalidation API.)

## 3. Cloudflare API tokens — not needed

- Because Launch **does not invoke the Cloudflare API**, customers **do not provide or manage Cloudflare API tokens** for Launch cache purging.
- Authentication for **Launch APIs** uses Contentstack's standard methods — **OAuth tokens or Authtokens**.

## 4. Multi-CDN cache invalidation (Cloudflare in front of Launch)

- Correct: the **Launch revalidation API applies only to Launch's own CDN layer.** If an external CDN (e.g. Cloudflare) sits **in front of** Launch, Launch-side invalidation **does not propagate** upstream.
- **Invalidation must be handled independently at each layer.** Two approaches:
  1. **Purge the upstream CDN** whenever content/deployments change (in addition to Launch's own invalidation).
  2. **Disable/minimize caching on the upstream CDN** and let **Launch manage caching + invalidation**.
- **Recommendation:** using an **external CDN in front of Launch is generally not recommended** — Launch already provides DNS management, caching, DDoS protection, and WAF. An extra CDN layer adds cache-management complexity, operational overhead, and potential performance impact.
- If an external CDN **is** used, the customer **owns its cache behavior**: Launch auto-purges its own cache on deploy, but any **Cloudflare-cached content must be purged separately**, and the external CDN **must forward the `Host` header** to Launch for correct routing.

## Generalized guidance (for future similar queries)

- **"Does Launch use/support Cloudflare APO?"** → **No.** Optimizations are Launch-native (Edge Functions, caching controls, Cache Priming).
- **"Does Launch call the Cloudflare API to purge cache?"** → **No.** Use the **Launch Cache Revalidation API** (paths / cache tags / hostnames), often automated via **Contentstack Automate**. No Cloudflare API tokens required; Launch APIs use OAuth/Authtokens.
- **"How do I invalidate cache with another CDN in front of Launch?"** → Launch invalidation **doesn't reach upstream**; either **purge the upstream CDN separately** on each change or **don't cache upstream** and let Launch own it. Prefer **not** layering an external CDN over Launch; if you do, **forward the Host header** and manage that layer's purges yourself.

## Status

- All questions answered; customer confirmed **no further questions**.
