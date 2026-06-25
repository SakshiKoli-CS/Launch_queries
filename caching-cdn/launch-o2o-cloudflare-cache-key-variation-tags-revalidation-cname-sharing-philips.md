QUESTION

Customer **Philips** (SF case **00053618**) is re-architecting their cache strategy after switching their own CDN from **Akamai → Cloudflare**, which now sits in front of **Launch's Cloudflare** (a **Cloudflare→Cloudflare / O2O** path). Akamai used to cache close to the consumer; Cloudflare caches at origin (Contentstack), so they're evaluating **moving cache control to the Launch side** and need answers on cache-key control, revalidation, and connecting their own CDN. Their setup: **80+ domains** hit **their** CDN first, which forwards to **a single Launch hostname** (`philips-prod.contentstackapps.com`) with the original host in a header **`x-original-hostname`**; the Next.js app derives locale from that, so the **same Launch URL must return different content/cache entries per original hostname**.

Questions: (Q1) influence **cache keys by a custom header** (`Vary`)? (Q2) **cache-tag purge via Automate** — ALL vs ANY tags, limits, speed? (Q3) **purge cache from inside the Next.js app** (not Automate)? (Q4) **Automate trigger usage/volume** counting. Plus: can they **bypass Contentstack's CDN and connect their own CDN directly**, use **Workers**, and **cache on their layer**; and **CNAME sharing** between zones.

_Keywords: O2O orange to orange, Cloudflare to Cloudflare, cache key variation custom header x-original-hostname Vary, separate contentstackapps.com per locale, cache tags purge ALL vs ANY, 100 cache tags limit, purge ~1 minute, revalidate-cdn-cache API from web app, Automate webhook purge, bypass Contentstack CDN, Cloudflare Workers reverse proxy, CNAME sharing DNS only grey cloud, cross-account security block, custom domains via API not supported._

ANSWER

## Q1 — Cache-key variation by custom header (`x-original-hostname` / `Vary`): NOT supported

- **Cache-key variation using custom headers (e.g. `x-original-hostname`) is not supported in Launch.** `Vary` alone won't give you separate cache entries per original hostname on a single Launch hostname.
- **Alternative:** use **separate `*.contentstackapps.com` domains per locale** so the **CDN cache key differs per locale** (independent caching, no conflicts), e.g. `philips-prod-ie.contentstackapps.com`, `philips-prod-de.contentstackapps.com`.
- **No extra cost** for adding multiple `*.contentstackapps.com` domains; the **domain limit can be raised** for those. **Charges apply only for custom hostnames** (e.g. `philips.com`).

## Q2 — Cache-tag purge via Automate: ANY-tag, max 100 tags, ~1 min

- Multiple cache tags in a purge request invalidate any resource associated with **at least one** of the tags (**ANY**, not ALL).
- **Max 100 cache tags per purge request.**
- Invalidation **typically completes within ~1 minute** of triggering.

## Q3 — Purge/revalidate from the web app (not Automate): yes, via the Launch API

- Call the **Revalidate CDN Cache API** directly:
  ```
  POST https://launch-api.contentstack.com/projects/PROJECT_UID/environments/ENVIRONMENT_UID/revalidate-cdn-cache
  ```
- Reference: Launch API → Revalidate CDN Cache. (Remember: **one parameter per request** — `cachePath` OR `hostnames` OR `cacheTags` — see [revalidate one-param entry](launch-revalidate-cdn-cache-one-parameter-per-request.md).)

## Can you bypass Contentstack's CDN / connect your own CDN directly? No — use O2O

- **Launch is tightly coupled to the Contentstack CDN (Cloudflare)** for caching, invalidation, and edge delivery — **no supported way to bypass it.** Bypassing would break cache-key logic, revalidation, Automate purges, and the revalidation APIs (all rely on the Contentstack CDN).
- **Supported pattern:** put **your CDN in front as a reverse proxy (O2O — Orange-to-Orange)**, but requests **still pass through Contentstack** to retain cache control. Ref: Launch go-live guide → Cloudflare Orange-to-Orange (O2O).

## O2O details — Workers + caching on the customer layer

- **Workers in front of Launch: yes.** In an O2O setup the customer's Cloudflare zone can use **Workers, Transform Rules, routing logic, header manipulation, edge redirects** — supported, aligns with O2O guidance.
- **Caching on the customer's CDN layer: possible, but needs careful config.** By default Launch's zone is the **origin CDN** and the **customer zone behaves as a reverse proxy**. To cache at the customer layer:
  1. Ensure responses stay **cacheable upstream** — Launch already sends `Cache-Control`, `s-maxage`, `stale-while-revalidate`.
  2. Configure the customer Cloudflare to **cache proxied responses** (Cache Rules / Workers; ensure proxy default doesn't bypass cache; explicitly enable caching of responses from the Launch hostname).
  3. **Design cache-key behavior carefully** — hostname/locale logic, path rules, header forwarding (e.g. `x-original-hostname`) — to avoid conflicts between the two Cloudflare layers.

## CNAME sharing between the customer zone and Launch zone — must be DNS-only (grey cloud)

- **CNAME share is possible from Launch's side** (e.g. route `countries.non-prod.cdn.philips.com` → `ph-nextjs-web-acceptance.eu-contentstackapps.com`).
- **Critical:** on the customer side the **CNAME must be set to DNS only (grey cloud), NOT proxied (orange cloud).** If proxied, **Cloudflare blocks the traffic due to cross-account security restrictions.**
- This cross-zone traffic is **blocked by default** (prevents unwanted traffic between customers) and is **unlocked with a backend setting on the Contentstack zone** to allow the specific customer source — no security impact since the traffic is expected.

## Custom domains via API — not supported (UI only)

- **Custom hostname/domain configuration is NOT available via Launch Public APIs** — it's done through the **Launch UI + domain-verification flow.** So the "separate Launch domain per locale" approach (≈400 domains at Philips' scale) currently requires **UI setup**. Feedback logged with the product team — **API domain management** is a natural need for large multi-domain setups.

## Generalized guidance (for future similar queries)

- **"Vary cache key by a custom header (e.g. original hostname)?"** → **Not supported.** Use **separate `*.contentstackapps.com` domains per locale** (free, raisable limit); custom hostnames cost extra.
- **Cache tags in a purge** = **ANY** match (not ALL), **≤100 tags/request**, **~1 min** to complete.
- **Purge from app code** = call the **Revalidate CDN Cache API** (one param per request); **Automate** is the webhook/event-driven alternative.
- **You cannot put your own CDN behind/instead of Contentstack's** — Launch is coupled to its Cloudflare CDN; use **O2O reverse proxy** (Workers/Transform Rules OK; customer-layer caching needs deliberate cache-key design).
- **Cross-zone CNAME sharing must be DNS-only (grey cloud)** — proxied (orange) is blocked by Cloudflare cross-account rules; Contentstack unlocks the specific source via a backend setting.
- Related: [Revalidate CDN Cache — one param/request](launch-revalidate-cdn-cache-one-parameter-per-request.md), [load-test 429 / Cache-Control](launch-loadtest-429-uncached-origin-cache-control-headers-moncler.md), [branded reverse proxy](../custom-domains-ssl/launch-branded-click-tracking-reverse-proxy-rewrites-requirements.md), [add domains / domain limit](../custom-domains-ssl/launch-add-new-domains-decommission-repurpose-environment.md).

## Status

- **Answered + ongoing.** Provided definitive answers to Q1–Q3 and the O2O/CNAME questions; **Q4 (Automate usage counting)** routed as an Automate-side question. Moving toward a **non-prod validation**: Contentstack provided a **temporary non-prod org/access**, **CNAME sharing** to be configured **DNS-only**, then **cache + functional testing** by Philips/Cloudflare. Calls scheduled; coordination with Cloudflare (CNAME sharing implementation) in progress.
