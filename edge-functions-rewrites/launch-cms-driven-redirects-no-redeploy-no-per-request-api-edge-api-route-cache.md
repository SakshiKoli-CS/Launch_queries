QUESTION

Customer **Sophos** (SF case **00052531**) wants to **manage URL redirects via the Contentstack UI** but is unhappy with both documented approaches:

- **Current:** hardcoded rules in `next.config.js` — stable/performant but **requires a code change + deployment per redirect update.**
- **Documented CMS approach** ([redirecting-urls](https://www.contentstack.com/docs/developers/how-to-guides/redirecting-urls)): extract slug from **every** request, **query the Contentstack API at runtime**, match against the redirect entry's "From" field.

Their concerns with the runtime-API approach: **latency on every request** (not just redirected ones), **unpredictable behavior** if the API errors/rate-limits, and **scalability limits** from per-second CDA rate limits.

**Requirement:** redirects **managed in the CMS UI**, applied at **CDN/edge level**, with **no runtime API call per request** AND **no redeploy per change** (redeploys **purge the whole site cache**, which is unacceptable for frequently-updated campaign redirects). They asked about **Cloudflare redirect rules** or a **Vercel-firewall-style** override.

_Keywords: CMS-driven redirects, manage redirects in UI no code deploy, no runtime API call per request, edge function redirects, next.config.js hardcoded redirects, build-time inject redirects, cms-driven-edge-redirects-rewrites, API route s-maxage cache redirects, automate webhook revalidate, redeploy purges site cache, Vercel firewall redirects, edge function cost per request before cache._

ANSWER

## Option A — build-time injection into an edge function (CMS-driven, no per-request API call)

- A Launch example fetches redirect entries from the CMS **at build time** via a script and **injects them into an edge function**, so redirects apply at the **CDN/edge with no per-request API lookup.** **Automation** redeploys the Launch project whenever a redirect entry is **created/updated/published** → fully CMS-driven.
- Repo: **`contentstack-launch-examples/cms-driven-edge-redirects-rewrites`**.
- **Limitation for this customer:** it **relies on redeploys**, and a **redeploy purges the whole site cache** — not viable when campaign redirects change frequently.

## Option B (recommended for "no redeploy + no per-request API") — API route with cache headers + edge function

To avoid **both** redeploys **and** a CMS call on every request:

1. **API route with cache headers** — a Next.js API route (e.g. `/pages/api/redirects`) that **fetches redirect/rewrite entries from Contentstack**, returned with **`Cache-Control` (e.g. `s-maxage`)** so the **CDN caches the response**. Refresh it either by:
   - **Automate / webhooks** to revalidate the cache when redirect entries change, **or**
   - a **shorter TTL** (e.g. **10–15 min**) so redirects refresh automatically.
2. **Edge function** (e.g. `/functions/[proxy].edge.js`) **fetches the rules from that API route** (forwarding the incoming request) and applies the redirect at the edge:
   ```js
   const response = await fetch(new Request(apiUrl, request));
   ```

**Benefits:** **no per-request CMS API call** (so no per-request latency / rate-limit exposure — the rules come from the **cached** API route), redirects are **CMS-driven and update without code changes/redeploys**, and freshness is handled by **cache TTL or automation.**

## Cost of running an edge function on every request

- Edge functions **do run on every request**, **but execute at the edge before the cache layer.** For **cached requests, traffic is served entirely at the edge and never reaches origin** — so adding the edge function **introduces no additional cost**, even at Sophos's traffic volume.

## Generalized guidance (for future similar queries)

- **"Manage redirects in the CMS, at the edge, without per-request API calls or redeploys"** → two patterns:
  - **Build-time injection** (`cms-driven-edge-redirects-rewrites`) — simplest, but **each change = redeploy** (which **purges site cache**). Fine if changes are infrequent.
  - **Cached API route + edge function** — edge function reads redirect rules from a **CDN-cached API route** (refreshed by **`s-maxage` TTL** or **Automate/webhook revalidation**); **no redeploy, no per-request CMS call.** Best for **frequently-updated** redirects.
- **Avoid the naive runtime-API-per-request pattern** for high traffic — it adds latency to every request and is exposed to **CDA rate limits**; cache the rules instead.
- **Edge functions run pre-cache** → **cached requests don't hit origin**, so an always-on edge function for redirects has **no extra origin/compute cost**.
- Note: **redeploys purge the entire site cache** — a key reason to prefer the cached-API-route approach when redirects change often.
- Related: [branded reverse proxy via launch.json rewrites](../custom-domains-ssl/launch-branded-click-tracking-reverse-proxy-rewrites-requirements.md), [cache priming](../caching-cdn/launch-cache-priming-exact-paths-no-wildcard-generate-launchjson-at-build.md), [Sophos CDN-over-CDN / HTTP/2](../caching-cdn/launch-cdn-over-cdn-akamai-caching-http2-tarpit-self-ddos-sophos.md).

## Status

- **Answered with two architectures.** Build-time-injected edge redirects (`cms-driven-edge-redirects-rewrites`) are CMS-driven but **redeploy-per-change** (purges cache). For **no redeploy + no per-request API**, recommended a **cached API route (`s-maxage` + Automate/webhook revalidation) read by an edge function**. Confirmed **no extra cost** from the always-on edge function (runs **pre-cache**; cached requests never reach origin). (Part of a broader Sophos escalation incl. HTTP/2 — see linked CDN-over-CDN entry.)
