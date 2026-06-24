QUESTION

Customer **Five Iron Golf** (SF case **00058416**) needed **branded click tracking** for **Braze** emails — clicks routing through `click.e.fiveirongolf.com` / `click.t.fiveirongolf.com` instead of going directly to the AWS/Braze tracking URL `r.us-east-1.awstrack.me`.

They first proposed having **Contentstack enable Cloudflare orange-cloud proxy** on `fiveirongolf.com` (since the Launch site is on Cloudflare-for-SaaS) to fix a **"Pending" hostname/SSL** state, asking whether Contentstack could flip those proxy records. Then they asked Launch to confirm it could act as a **reverse proxy** for the branded click domains, with **7 specific requirements**.

_Keywords: branded click tracking, Braze custom click domain, awstrack.me, reverse proxy Launch, Cloudflare orange cloud proxy, hostname pending SSL, launch.json rewrites reverse proxy, preserve path query string, pass through redirects, disable caching, no WAF/bot/JS challenge, click volume, Cloudflare for SaaS._

ANSWER

## On the Cloudflare orange-cloud proxy request → not the recommended path

- **Proxying the domain through Contentstack's Cloudflare account (orange cloud) is not recommended and may not be possible directly.** Don't pursue flipping proxy records in Contentstack's Cloudflare zone — use one of the supported approaches below instead.

## Two supported approaches for branded click tracking

1. **Native Braze custom click-tracking domain (cleanest):** add a **CNAME at your DNS provider** pointing to the endpoint Braze provides, then enable the custom domain in Braze — **SSL managed by Braze**. Confirm with Braze whether the native custom-domain option is on your plan.
   - Customer note: Braze has no default click domain (all customers use custom domains); registering a dedicated click domain (e.g. `click-fiveirongolf.com`) raises **deliverability** (Google/Microsoft/Yahoo AI filtering) and **brand-legitimacy** considerations to weigh.
2. **Reverse proxy via Launch (Edge URL Rewrites / Edge Functions):** Launch hosts the branded domain and **transparently proxies** to the Braze/AWS tracking endpoint. Use **`launch.json` rewrites** for a simple high-performance proxy, or **Edge Functions** for advanced routing.
   - Example `launch.json` rewrite:
     ```json
     {
       "rewrites": [
         { "source": "/:path*", "destination": "https://r.us-east-1.awstrack.me/:path*" }
       ]
     }
     ```
     Any request on the branded domain is proxied to the tracking endpoint, preserving the full path + query.

## Launch reverse-proxy — point-by-point confirmation (all YES)

1. **Host SSL** for `click.e.` / `click.t.fiveirongolf.com` → Launch supports custom domains and **auto-provisions/manages SSL** for them (HTTPS).
2. **Route to `r.us-east-1.awstrack.me`** → Launch can proxy requests to **external origins**, including the AWS/Braze endpoint.
3. **Preserve full path + query string** → rewrite rules forward path/query **unmodified** (campaign IDs, user IDs, params intact).
4. **Pass through redirects (301/302/etc.) without rewriting/caching** → Launch forwards upstream redirects transparently to the browser.
5. **Disable caching for these routes** → configurable via response headers; set these routes **non-cacheable** so every click is processed in real time.
6. **No bot/WAF/auth/JS challenges on click paths** → configure these routes **without** added auth/bot-protection/JS challenges (they'd interfere with email link validation).
7. **Support email-campaign click volume** → Launch runs on a globally distributed CDN built for **high-traffic bursts**.

**Recommendation:** validate the full setup in a **lower/non-production environment first** — verify redirect handling, cache-control, and Braze deliverability expectations before production.

## Generalized guidance (for future similar queries)

- **"Can Launch reverse-proxy our branded click/tracking domain to Braze/AWS (awstrack.me)?"** → **Yes.** Host the branded domain on Launch (auto SSL), and proxy to the tracking endpoint via **`launch.json` rewrites** (`/:path*` → `https://<endpoint>/:path*`) or **Edge Functions**.
- **Must-haves for click-tracking proxy routes:** preserve path/query, **pass redirects through untouched**, **disable caching** (`no-store`), and **don't apply WAF/bot/JS challenges**.
- **Don't** ask Contentstack to flip **Cloudflare orange-cloud proxy** records — not recommended/feasible; use the rewrite/Edge-Function proxy or native Braze custom domain instead.
- **Prefer native Braze custom domain** when available (Braze manages SSL); weigh **deliverability/branding** of a separate click domain.
- Test in **non-prod** first. Related (HTTPS cert-mismatch + Next.js catch-all proxy variant): [launch-ses-click-tracking-custom-domain-https-proxy.md](launch-ses-click-tracking-custom-domain-https-proxy.md).

## Status

- Launch confirmed it **can** support the reverse-proxy use case point-by-point (rewrites/Edge Functions). Customer also evaluating native Braze custom domain + deliverability. **Ticket closed** (no further customer response); reopen if they return.
