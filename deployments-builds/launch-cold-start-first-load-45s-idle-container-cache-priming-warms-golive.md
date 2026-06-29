QUESTION

Customer **High Street Partners** (via HUGE, SF case **00050755**; sites `www.myhighstreet.com`, `hsip-partners-production.azcontentstackapps.com` → later `www.highstreetpartners.com`) reports a Launch issue: the **Node instance shuts down when there's no traffic**, so the **first load takes ~45+ seconds** (cold start). Sites are **soft-launched**. What needs coordinating?

_Keywords: Node instance shuts down no traffic, cold start first load 45 seconds, idle container, cache priming warms containers, cache-control headers required for priming to cache, soft launch, add apex domains alongside www, TXT DCV records hostname validation cert, go-live A records CNAME engineer call._

ANSWER

## Cold start — expected on first request to a freshly initialized container

- The **~45s first load is a cold start** — it occurs on the **first request to a newly initialized container.** This **should not happen often once the site is live and receiving steady production traffic.**
- Cold starts **may occur during redeployment**, but the customer has **cache priming configured**, which **warms containers even before the site is live** (the priming pass hits the pages). So in practice, steady traffic + cache priming keeps it from being a recurring problem.

## Why some cache-priming URLs aren't getting cached

- **Cache Priming just fetches the response at the edge/CDN before the site is live.** **Whether the response actually gets cached depends on the page having appropriate `Cache-Control` headers.**
- So **priming URLs that "aren't caching as expected" usually lack `Cache-Control` headers** on the page response — add them and priming will cache as intended.

## Go-live domain steps (apex + www)

- The customer has added the **`www` subdomains** in Launch. They should **also add the apex domains** to their environments (per the Custom Domains docs) if not already.
- Adding the domains yields the **TXT + DCV records** to add at their **DNS provider** for **hostname validation + cert provisioning** (for both apex and www). Status shows on the **Domains tab** in project settings.
- **At go-live**, a **Launch engineer joins the call** to guide the final step: **add the A records + CNAME records** at the DNS provider, which **moves all traffic to the new Launch-hosted site.**

## Generalized guidance (for future similar queries)

- **"Node instance shuts down with no traffic → slow (~45s) first load"** → expected **cold start** on idle containers; **resolves with steady production traffic.** **Cache Priming warms containers** (even pre-launch) to mitigate it, including across redeploys.
- **Cache-priming URLs not getting cached** → the page response is **missing `Cache-Control` headers**; priming only *fetches* at the edge — **caching requires the right cache-control headers.**
- **Go-live domain flow:** add **apex + www** in Launch → add **TXT/DCV** records at DNS for **validation + cert** → at cutover add **A + CNAME** records (Launch engineer guides). Statuses on the **Domains tab**.
- Related: [idle environment Warm-Up Screen / keep warm](../logs-monitoring/launch-idle-environment-sleep-error-warm-up-screen.md), [cold start + health-check warming at go-live](launch-npm-err-invalid-arg-type-path-null-security-fix-revert-sso-ssr-cache-cold-start.md), [cache priming exact-paths / build-time gen](../caching-cdn/launch-cache-priming-exact-paths-no-wildcard-generate-launchjson-at-build.md), [live go-live support / apex self-serve](launch-live-go-live-support-coverage-apex-now-self-serve-ui.md).

## Status

- **Explained.** The **~45s first load = cold start** on idle containers; **eases with steady traffic**, and **cache priming warms containers** (even pre-live / across redeploys). **Cache-priming URLs not caching** → add **`Cache-Control` headers** to those page responses. Go-live: add **apex + www** in Launch, complete **TXT/DCV** validation + certs, then **A/CNAME** at cutover with a **Launch engineer on the call.**
