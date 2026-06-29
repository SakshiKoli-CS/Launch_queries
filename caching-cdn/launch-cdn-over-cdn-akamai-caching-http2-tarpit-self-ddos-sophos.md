QUESTION

Multi-topic **Sophos** escalation (Launch + Akamai in front). Three distinct concerns discussed on a call:

1. **HTTP/2 protocol errors** for users.
2. **Caching at Akamai instead of Launch (Cloudflare)** — customer wants caching on **their** CDN (Akamai) for better **cache analytics visibility** they don't get on Launch.
3. A recent **self-induced DDoS** — huge origin traffic spikes (Jan 7 ~76M requests, Jan 9 ~85M requests, mostly **404s**) causing **thousands of dollars/day** in compute.

_Keywords: HTTP/2 protocol error, WAF Tarpit JS detection failed, Akamai header size, CDN over CDN caching Akamai over Cloudflare Launch, cache analytics visibility, automate Akamai purge on deploy, cache-control max-age avoid, self-induced DDoS 404 spike origin compute cost, CDN-over-CDN delays DDoS detection, CDA detected Launch did not._

ANSWER

## 1. HTTP/2 protocol errors — caused by a WAF Tarpit rule (NOT header size)

- **Resolved cause (customer-confirmed):** the HTTP/2 protocol errors were **unrelated to header size**. They were caused by a **WAF rule set to Tarpit when JS detection failed.** After adjusting it, **no repeat** for users.
- Note for history: an earlier call had floated **smaller Akamai response-header limits** as a *possible* cause and suggested **reviewing Akamai logs purely as a debugging step** — not as blame. The real cause turned out to be the **WAF Tarpit-on-JS-detection-failure** rule.

## 2. Caching at Akamai (CDN-over-CDN) — possible, but understand the risks

- The customer wants to cache at **Akamai** (their CDN) rather than **Launch's Cloudflare** to get **cache analytics** they lack on Launch.
- This is a **CDN-over-CDN** setup with real risks; key mitigations:
  - **Stale content after deploys:** when a new deployment goes live and **Akamai's cache isn't cleared**, some pages can **intermittently break for a few minutes**. **Automate Akamai cache purges during deployment** (via Automate) to avoid this.
  - **Avoid `Cache-Control: max-age`** (which their site currently uses) — `max-age` drives **browser** caching and can **prevent content updates from reflecting**. Use **`s-maxage`** for CDN-layer control instead. (Standard best practice, not tied to a specific incident.)
  - **CDN-over-CDN can delay DDoS detection** (see #3).

## 3. Self-induced DDoS — 404 flood, high origin compute cost; CDN-over-CDN delayed detection

- **Jan 7 (~76M requests)** and **Jan 9 (~85M requests)**, **mostly 404s**, hit the origin → **thousands of dollars/day** in compute.
- Customer indicated it was likely **their own scripts/tools** and said they'd **block that traffic.**
- **Key architectural insight:** the **CDN-over-CDN setup delayed DDoS detection** — **Launch (Cloudflare) did not detect it**, while **CDA did** for their traffic (the CDA team had already flagged ~50M requests). Putting another CDN in front can **blunt Cloudflare's native DDoS protections.**

## Generalized guidance (for future similar queries)

- **HTTP/2 protocol errors** → don't assume header size; a common real cause is a **WAF rule (e.g. Tarpit on failed JS/bot detection)** mis-firing on legitimate users. Check **WAF/bot rules** first.
- **"We want to cache on our own CDN (Akamai) in front of Launch for analytics"** → it's a **CDN-over-CDN** pattern; warn on: (a) **stale pages after deploys** unless you **automate the upstream CDN purge on deploy**, (b) **don't use `Cache-Control: max-age`** (browser caching blocks updates) — use **`s-maxage`**, and (c) **delayed DDoS detection** because the outer CDN hides traffic from Cloudflare's protections.
- **Sudden 404 flood + huge origin compute cost** → likely **self-induced (customer scripts/tools)**; have them **block the source**. In a **CDN-over-CDN** setup, expect **Cloudflare/Launch to under-detect** it — corroborate with **CDA** traffic data.
- Related: [O2O Cloudflare cache architecture](launch-o2o-cloudflare-cache-key-variation-tags-revalidation-cname-sharing-philips.md), [stale published content / max-age vs s-maxage](launch-stale-published-content-cdn-cache-no-cache-control-headers-next.md), [403 programmatic access — WAF/Bot](../networking-connectivity/launch-403-programmatic-access-cloudflare-waf-bot-browser-integrity-check.md), [load-test 429 / Cache-Control](launch-loadtest-429-uncached-origin-cache-control-headers-moncler.md).

## Status

- **HTTP/2 errors: resolved** — **WAF Tarpit-on-JS-detection-failure** rule (not header size); no repeat after fix. **Akamai caching (CDN-over-CDN):** advised on risks + mitigations (**automate upstream purge on deploy**, **use `s-maxage` not `max-age`**, **delayed DDoS detection**). **DDoS: self-induced 404 flood** (Jan 7/9, ~76M/~85M req), high origin cost; **CDN-over-CDN delayed Launch/Cloudflare detection** (CDA detected); customer to block the traffic. Action items: customer continues HTTP/2 investigation with Akamai; Launch to share **Jan 7/9 + a normal-day** traffic reports for comparison.
