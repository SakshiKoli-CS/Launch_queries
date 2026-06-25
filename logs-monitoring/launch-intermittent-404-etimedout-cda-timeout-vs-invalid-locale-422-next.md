QUESTION

Customer **Next Retail Limited** (Org `blte250adf4396ee460`, Stack `blt9c9c23f853325239`, Azure EU, SF case **00053262**, **P1**, JIRA **CL-2781**) attempted go-live and **multiple users hit "Page Not Found"** — had to roll back. App shows **"No content found for: /desktop/en-mx"** (the react-ssr-fragment-app baseline returns Page Not Found when CDA returns null).

- The content **exists** and **direct CDA API requests succeed**; through the frontend it's **intermittent** (fails, then succeeds minutes later). Seen on both Launch subdomains (`staging-…` / `production-next-international.eu-azcontentstackapps.com`).
- **Redeploying the frontend temporarily cleared it** — not viable for a live site.
- Launch **Server Logs** showed only:
  ```
  [ORIGIN] Error fetching page "/desktop/en-mx": [APIError: Error] { error_code: 'ETIMEDOUT', status: 0, error_message: 'Error' }
  ```
- Errors occur **under very low load** (avg ~0.112s, median ~0.044s response), recur in **short windows**, and **self-resolve in ~5–10 min**.

_Keywords: Page Not Found go-live, No content found locale, ETIMEDOUT status 0 origin, CDA API timeout intermittent, react-ssr-fragment-app returns null 404, invalid locale 422 Language was not found, environment not found, low load timeouts, redeploy temporarily fixes, default domain direct access link clicks, block default domain edge function, cache headers s-maxage, CL-2781._

ANSWER

This thread had **two intertwined signals** — keep them separate.

## Signal 1 (the real "Page Not Found") — intermittent CDA `ETIMEDOUT` on the SSR→CDA call

- The frontend's **SSR call to the CDA timed out** (`ETIMEDOUT, status: 0`), so the app received **null → rendered "No content found / Page Not Found"** even though the content exists and **direct CDA calls succeed.**
- It's **intermittent**, **self-resolves in minutes**, and happens **under low load** — so the customer correctly pushed back that **retries/backoff or a bigger SSR timeout isn't the real fix**; the question is **why CDA times out at low load.**
- **`status: 0` / `ETIMEDOUT`** = the request **never got a usable response at the transport level** (connection/timeout), not an HTTP error from CDA. CDA-side **origin/CDN logs showed no 5xx and no matching timeout** in the windows → the failure is on the **network path / SSR→CDA connection**, consistent with the earlier [TCP `ETIMEDOUT`/autoscaling pattern](../networking-connectivity/launch-intermittent-tcp-etimedout-econnrefused-cda-autoscaling-spike.md).

## Signal 2 (noise, NOT the cause) — invalid-locale / invalid-environment `422`s

- CDA logs showed many **`422` "Language was not found"** (and environment-not-found) for paths like `fonts`, `entry-client-*.css`, `*.js`, `favicon.ico`, `shop`, `brands` — these are **the app treating non-page paths (assets, relative links) as locale page-entry requests.**
- **Source:** users can open the **Launch default domain directly** (or via **Live Preview**) and **click relative links**, which the frontend then treats as **page-entry CDA requests** with an invalid "locale." These **422s are real but are NOT what causes the `ETIMEDOUT` failures on valid requests** — confirmed in-thread.
- Don't let the high-volume 422 noise mask Signal 1.

## What actually helped (customer-side mitigations)

The customer reported the original issue stopped after they:
1. **Added cache headers** on Production Launch envs: **`public, max-age=0, s-maxage=60`** (CDN serves pages, fewer SSR→CDA calls).
2. **Filtered non-content paths before they reach CDA** — skip **`/api`, `/fonts`, `favicon.ico`, `chrome.devtools.json`**, etc., so assets/relative-link clicks don't become bogus CDA entry requests.

Additional recommendations:
- To stop **direct default-domain access / link-clicking** generating bogus requests, **block the default domain via an Edge Function** (allow only intended domains): [launch-edge-default-domain-blocking example](https://github.com/contentstack-launch-examples/launch-edge-default-domain-blocking).
- Verify pages are **published to the correct locale + environment** (customer confirmed they were, and a direct CDA call in the same format succeeded — which is why this is a **timeout**, not a publish gap).

## Generalized guidance (for future similar queries)

- **"Page Not Found / No content found" intermittently at go-live, but content exists and direct CDA works** → the **SSR→CDA fetch is timing out (`ETIMEDOUT, status 0`)** and the app renders null as a 404. **Not a content/publish problem; not retries-will-fix.** Investigate the **network path / connection to CDA** (matches the TCP-timeout/autoscaling class).
- **`status: 0` means transport-level failure** (no HTTP response) — look at connectivity/timeouts, not CDA HTTP error codes. CDA origin logs showing **no 5xx** is consistent with this.
- **Separate the 422 invalid-locale noise from the timeout.** **422 "Language was not found"** on asset/relative paths = the app mis-routing **non-page requests (assets, default-domain link clicks, Live Preview)** as locale page-entry calls. Fix by **path-filtering before CDA** and **blocking the default domain** — but know it's **not** the cause of the valid-request timeouts.
- **Mitigations that reduce SSR→CDA pressure:** **cache headers** (`s-maxage`) so the CDN serves pages, and **filtering non-content paths**. A **redeploy "fixing" it temporarily** is a symptom of a transient/connection issue, not a real resolution.
- Related: [TCP ETIMEDOUT/ECONNREFUSED to CDA — autoscaling](../networking-connectivity/launch-intermittent-tcp-etimedout-econnrefused-cda-autoscaling-spike.md), [load-test 429 / Cache-Control](../caching-cdn/launch-loadtest-429-uncached-origin-cache-control-headers-moncler.md). (To block direct default-domain access: the [launch-edge-default-domain-blocking example repo](https://github.com/contentstack-launch-examples/launch-edge-default-domain-blocking).)

## Status

- **Mitigated / no longer reproducing.** Root signal: **intermittent CDA `ETIMEDOUT` on the SSR→CDA call** (low load; CDA origin logs showed no 5xx/timeout → network-path class). Customer added **`s-maxage=60` cache headers** + **path filtering** (skip `/api`, `/fonts`, `favicon.ico`, etc.); **stopped seeing the ETIMEDOUT** afterward. The **422 invalid-locale** entries were **separate noise** (default-domain/relative-link/Live-Preview misrouting), addressable via path filtering + **default-domain blocking Edge Function**. Tracked **CL-2781**.
