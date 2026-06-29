QUESTION

Customer **Drive Automotive** (Org `blt1e31192a48436504`, GCP NA) — proactively flagged by support — has **timeout/slowness** across several sites (`prideautocare.com`/`www`, `kerryscarcare.com`, `servicestreet.com`; same codebase). Two observations:

1. The DNS for `prideautocare.com` is pointed to **`pride-auto-care-dev.gcpcontentstackapps.com`** — a **dev environment**. Is that correct?
2. Application logs show **504 Bad Gateway** errors, suspected to drive the elevated timeouts.

Customer then asked Launch to investigate the errors and how to **improve site load time.**

_Keywords: 504 Bad Gateway timeout, prod domain points to dev environment gcpcontentstackapps, Reputation.com 504 Gateway Timeout, HeadersTimeoutError, backend service unavailable not Launch, CMS fetch timeout 408 Timeout of undefinedms, no non-200 in origin CDN logs, improve load time performance cache priming rendering strategies._

ANSWER

## The 504s are from the customer's BACKEND, not Launch

- The **504 Bad Gateway** errors originate from the **application's own backend calls**, not Launch. Concrete log evidence:
  ```
  [cause]: Error [HeadersTimeoutError]: Headers Timeout Error
  Error fetching reviews from Reputation.com: Failed to fetch reviews: 504 Gateway Timeout
  Status Code: 504
  ```
- i.e. the app calls a **third-party service (Reputation.com)** that returns **504** → surfaces as site timeouts. **Launch can't debug this** — the customer must investigate the **backend service (Reputation.com)** for the latency/downtime.
- **No platform incident**: CMS/CDN-side checks (origin + Cloudflare/Fastly CDN logs over 7 days) found **no non-200 responses** for the org.

## Prod domain pointing to a dev environment — confirmed intentional

- `prideautocare.com` resolving to **`pride-auto-care-dev.gcpcontentstackapps.com`** (a dev env) was **flagged** — but the customer **confirmed it's intentional / functional** and chose to keep it. **Not the cause** of the timeouts. (Still worth flagging on any audit, as prod→dev pointing is unusual.)

## Separate, transient CMS-fetch timeouts (different from the backend 504s)

- Distinct from the backend 504s, logs showed **two brief CMS entry-fetch timeouts** during page load (did not persist):
  ```
  Error fetching brand entry with UID …: {"error_message":"Timeout of undefinedms exceeded","error_code":408}
  Error fetching service entries: {"error_message":"Timeout of undefinedms exceeded","error_code":408}
  ```
- These are **transient `408` timeouts fetching CMS stack entries** during render — occurred **twice, not persistent**; raised with the CMS team (no broader pattern found).

## Improving site load time (performance recommendations)

For the "make the site faster" ask:
- **Cache Priming** — preload frequently-accessed pages to the CDN at deploy (needs proper cache-control headers).
- **Mixed rendering** — SSG / SSG+priming / SSR+CDN caching / SSR+priming as appropriate.
- **Leverage CDN caching** via correct **`Cache-Control`** so content serves from POPs; use **Automate** to **selectively revalidate** changed pages instead of full redeploys.
- **Observability** — export server logs to a **central log target**; alert on **latency spikes, error rates, cache-hit-ratio drops.**
- (Caching Guide / Cache Priming / Revalidate-via-Automate / Log Targets docs.)

## Generalized guidance (for future similar queries)

- **Site 504 / timeout, "is Launch down?"** → check **where the 504 originates.** If logs show a **third-party/backend call** (e.g. `Reputation.com`, `HeadersTimeoutError`) returning 504, it's **the customer's backend**, not Launch — verify with **origin + CDN logs showing no non-200** platform-side. Customer must debug their backend.
- **Distinguish** backend 504s from **transient CMS-fetch `408` timeouts** (`Timeout of undefinedms exceeded`) — the latter are intermittent entry-fetch timeouts at render, escalate to CMS if persistent.
- **Prod domain pointing to a `*-dev.*contentstackapps.com` env** is unusual — flag it, but it can be **intentional/functional**; confirm before assuming it's the bug.
- **"Improve/reduce load time"** → Cache Priming + mixed rendering + CDN cache-control + Automate selective revalidation + observability/alerting.
- Related: [Cloudflare 504 downtime (proactive guard)](launch-downtime-cloudflare-504-retry-redeploy-proactive-guard.md), [intermittent 404/ETIMEDOUT SSR→CDA](launch-intermittent-404-etimedout-cda-timeout-vs-invalid-locale-422-next.md), [scalability/perf (cache priming, rendering)](../deployments-builds/launch-scalability-compute-limits-large-ssg-builds-runtime-throughput-sla.md).

## Status

- **Explained — not a Launch issue.** The **504s are the app's backend (Reputation.com) timing out** (`HeadersTimeoutError` / 504); **no non-200 in Launch origin/CDN logs.** **Prod→dev domain pointing confirmed intentional** by customer (not the cause). Two **transient CMS-fetch `408` timeouts** noted separately (non-persistent, raised with CMS). Shared **performance recommendations** (cache priming, rendering strategies, CDN caching, Automate revalidation, observability) for the load-time ask.
