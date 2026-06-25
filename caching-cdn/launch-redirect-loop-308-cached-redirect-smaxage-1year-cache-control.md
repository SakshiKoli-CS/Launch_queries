QUESTION

Customer **Bibby Financial Services** (via iO Group NV; Org `blt7ba943695da4045e`, Launch project "Bibby web project", **Production**, domain `bibbyfactor.fr`, SF case **00053455**) reports the **homepage is stuck in a redirect loop** — root URL `https://www.bibbyfactor.fr/` keeps **redirecting to `/`** (**HTTP 308**, "too many redirects" in the browser).

- The **redirect response appears to be cached by Cloudflare** (`Server: Cloudflare`).
- **Only this domain is affected**; other Bibby URLs/domains are fine.
- The customer **has no direct Cloudflare access** and can't clear the cache themselves.
- They say their app **doesn't set a 1-year cache header** and **doesn't set a `/` → `/` redirect**; they use an **Edge Function driven by an uploaded Excel file** (redirects for various sites, but **not** the homepage).

_Keywords: redirect loop, too many redirects, 308 status, cached redirect Cloudflare, Server Cloudflare, s-maxage 31536000 one year, cache-control mismatch, homepage / to / redirect, edge function redirects excel, no-store redirects, purge cached redirect CDN, redirection misconfiguration._

ANSWER

## Immediate fix — manually purge the Cloudflare cache

- Support **manually cleared the Cloudflare cache** for the domain → site recovered immediately. (Customer can't do this themselves; they have no Cloudflare access.)

## Why it happened — a bad redirect got cached at the CDN for ~1 year

- The domain's response carried a **Cache-Control that caches at the CDN for a full year**:
  - **`bibbyfactor.fr`:** `public, max-age=0, s-maxage=31536000, must-revalidate` → **`s-maxage=31536000` = 1 year** at Cloudflare.
  - **Other Bibby apps:** `public, max-age=0, s-maxage=300, stale-while-revalidate=60` → CDN caching limited to **5 minutes**, avoiding long-lived issues.
- So when a (mis)configured **`/` → `/` redirect (308)** was produced, the CDN **cached that redirect for a year**, turning a transient misconfig into a persistent **redirect loop**. The final root cause was a **redirection configuration issue** (fixed), but the **1-year `s-maxage` made it sticky**.

## Recommendations (prevent recurrence)

1. **Don't long-cache redirects.** Use a **low `s-maxage`** or **disable caching for redirects**, e.g.:
   - `Cache-Control: no-store`, or
   - `public, max-age=0, s-maxage=300, stale-while-revalidate=60`
   - (Adding `no-store` to each redirect in the Edge Function is good for completeness, but the **real fix was the redirect config + the 1-year cache header**.)
   - Docs: Cache-Control headers.
2. **Purge cached redirects from the CDN** when one slips through — selectively via **time-based**, **path-based**, or **tag-based** purging (cache tags). Docs: cache purging using cache tags.

## Generalized guidance (for future similar queries)

- **"Redirect loop / too many redirects (308) on one domain, `Server: Cloudflare`"** → a **redirect got cached at the CDN**. **Immediate unblock: purge the Cloudflare cache** (support-side; customer usually can't). Then find **why the redirect was emitted** AND **why it stuck**.
- **Check the `Cache-Control` `s-maxage`.** A huge value like **`s-maxage=31536000` (1 year)** on an HTML/redirect response will **pin a bad redirect at the CDN indefinitely**. Compare against healthy siblings (here, others used `s-maxage=300` + `stale-while-revalidate`).
- **Never long-cache redirects** — use `no-store` or a small `s-maxage`. A 1-year cache turns a momentary misconfig into a persistent outage.
- **"Our app doesn't set that header / that redirect"** — still verify the **emitted response headers** and any **Edge Function / redirect rules**; the effective `Cache-Control` and redirect can come from framework defaults, the edge layer, or config, not just explicit app code. Root cause here was a **redirection configuration issue**.
- Related: [SES/branded redirect caching](launch-branded-click-tracking-reverse-proxy-rewrites-requirements.md) (no-cache on pass-through), [Revalidate CDN Cache / purge](launch-revalidate-cdn-cache-one-parameter-per-request.md).

## Status

- **Resolved.** Unblocked by **manually purging the Cloudflare cache**; recurred briefly, then **fixed by correcting the redirection configuration**. Root issue: a **misconfigured `/`→`/` 308 redirect** that was **cached for ~1 year via `s-maxage=31536000`**. Recommended **short/no caching for redirects** + **CDN purge** to prevent recurrence.
