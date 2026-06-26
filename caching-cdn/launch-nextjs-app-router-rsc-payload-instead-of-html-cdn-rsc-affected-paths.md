QUESTION

Customer **MicroStrategy / Strategy** (SF case **00053122**) reports that **certain routes intermittently don't load on Launch** and display **wrong/garbled content** (an **RSC payload** instead of the page) — e.g. `https://community.strategy.com/security-hub`, `/documentation`, `https://community.contentstackapps.com/security-hub`.

- **Intermittent**, **specific routes**, **no recent deployment**, **env vars fine**, framework **Next.js 15.5.9 App Router**.

Scalability follow-up: they have **lots of dynamic pages whose URLs are driven by Contentstack entries**, so **listing every path** in the mitigation script "isn't scalable." They ask: **wildcard/pattern support** in the affected-paths list? a **global/catch-all**? can it be handled at **CDN/Launch config** instead of in code? and **exactly what `Cache-Control` value** avoids the issue?

_Keywords: Next.js App Router RSC payload instead of HTML, React Server Components behind CDN, wrong content intermittent routes, RSC_AFFECTED_PATHS script, redirects in next config trigger RSC, handling nextjs rsc issues on launch, wildcard catch-all affected paths, cache-control no-store must-revalidate, CDN caches redirect intermediate state, dynamic CMS pages scalability._

ANSWER

## Cause — Next.js App Router returns RSC payloads that the CDN caches/serves instead of HTML

- This is a **known Next.js App Router (React Server Components)** behavior: running an App Router app **behind a CDN** (the Launch CDN), the app can **return RSC payloads instead of the expected HTML page**, and the CDN may **cache that intermediate/redirect state**, so subsequent requests get the **wrong content**.
- It's a **framework-level class of issue** (Next.js + CDN + redirects), not a Launch app/config bug. Official handling: [Handling Next.js RSC issues on Launch](https://www.contentstack.com/docs/developers/launch/handling-nextjs-rsc-issues-on-launch).

## Mitigation 1 — the RSC-affected-paths script

- Add affected routes to the **`RSC_AFFECTED_PATHS`** list per the doc so the correct content is consistently served **while still allowing CDN caching** (keeps the performance/latency benefit).
- **Redirected routes count too:** new pages where it appeared were **routes coming from redirects in the Next.js config** — those **also resolve through the App Router**, so they can trigger the **same RSC behavior** and need covering.

## Mitigation 2 — `Cache-Control` to skip CDN caching on affected routes (brute-force, scalable)

- For the customer's **"thousands of CMS-driven dynamic URLs — listing each path isn't feasible"** concern, the conservative fix is to **prevent CDN caching on the affected pages/routes** via a header:
  ```
  Cache-Control: no-store, must-revalidate
  ```
  This is the **direct answer to "what value avoids the issue"** — it's **not** time-based or on-demand revalidation; it simply **stops those routes being cached at the CDN**, avoiding the RSC mix-up entirely **at the cost of reduced caching efficiency** on those routes.

## On the scalability questions (wildcards / catch-all / CDN-level)

- The recommended, supported handling is the **doc's script + `RSC_AFFECTED_PATHS`** and/or the **`no-store` cache-control** approach. **Any route that can return an RSC response (including redirected ones) may need to be covered.**
- For **CMS-heavy sites with many dynamic routes**, the **`Cache-Control: no-store, must-revalidate`** approach is the practical catch-all alternative (apply to the affected route patterns) since it **avoids the issue without enumerating every URL** — trading away CDN caching on those routes.

## Generalized guidance (for future similar queries)

- **"Next.js App Router site intermittently serves RSC payload / wrong content behind the CDN"** → known **RSC-behind-CDN** issue (worse with **redirects**, which also flow through the App Router). Follow the **Handling Next.js RSC issues on Launch** doc.
- **Two mitigations:** (1) **`RSC_AFFECTED_PATHS` script** — keeps CDN caching, but you must cover affected routes **including redirected ones**; (2) **`Cache-Control: no-store, must-revalidate`** on affected routes — **avoids it entirely**, scalable for many dynamic CMS URLs, **at the cost of CDN caching** on those routes.
- The **exact header to stop the behavior is `no-store, must-revalidate`** (not time-based/on-demand revalidation).
- Related: [redirect loop cached 1 year / cache-control](launch-redirect-loop-308-cached-redirect-smaxage-1year-cache-control.md), [load-test 429 / Cache-Control](launch-loadtest-429-uncached-origin-cache-control-headers-moncler.md).

## Status

- **Explained + mitigations provided.** Cause: **Next.js App Router RSC payloads cached/served by the CDN** (intermittent, framework-level, aggravated by **config redirects** which also hit the App Router). Fixes: **`RSC_AFFECTED_PATHS` script** (covers routes incl. redirects, keeps caching) or **`Cache-Control: no-store, must-revalidate`** on affected routes (scalable catch-all for dynamic CMS pages, less caching). Awaiting customer confirmation.
