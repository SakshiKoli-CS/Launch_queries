QUESTION

Customer **Philips** asked about **Launch cache priming**:
1. What happens to **failing pages** that are primed — does it **fail the build**?
2. How **aggressive** is priming with respect to **CDA rate limits** — does it consider rate-limit HTTP headers?
3. (Follow-up) **How fast** does it request the URLs — all at the same time?

_Keywords: cache priming, primed page fails build, does priming fail deployment, cache priming rate limit, CDA rate limits, rate-limit headers, priming all at once, batches, warm CDN cache, preload URLs._

ANSWER

## 1. Failing primed pages do NOT fail the build/deployment

- **No** — if some pages fail during cache priming, the **build/deployment still succeeds.**
- Cache priming is a **performance optimization** that runs during deployment to **preload selected URLs into the CDN cache** (improve performance / reduce latency). It is **not** a gate on deployment.
- If some pages **timeout or error**, the process **continues for the rest** — nothing blocks the deployment. Failures are **logged**.

## 2. Rate limits — priming doesn't call CDA directly

- **Cache priming does not call CDA APIs directly.** It works by **requesting the Launch-deployed URLs** to warm the **CDN cache**.
- Any **CDA calls made by the application** (e.g. during **SSR**) still **follow CDA rate limits** — but **cache priming itself does not dynamically adjust** based on CDA rate-limit headers.
- Caching is handled at the **CDN level via HTTP cache headers**; revalidation via **cache-control headers** or **on-demand triggers**.

## 3. Request pacing — controlled batches, not all at once

- Priming **does not request all URLs simultaneously.** It sends requests in **controlled batches**, warming URLs **gradually**.
- Failures are **logged** and the process **continues** — designed to be **fast but safe**, without **overloading the site**.

Docs: Cache Priming in Launch, Caching Guide for Launch, Launch FAQs.

## Generalized guidance (for future similar queries)

- **"Will a failing primed page break my deploy?"** → **No** — priming failures are logged and skipped; deployment still succeeds (priming is a non-blocking performance step).
- **"Does priming respect CDA rate limits / rate-limit headers?"** → priming hits **deployed URLs** (CDN warming), **not CDA directly**, and **doesn't auto-throttle** on CDA rate-limit headers; the app's own SSR/CDA calls still obey CDA limits.
- **"Does it hammer the site (all URLs at once)?"** → No — **controlled batches**, gradual, won't overload.

## Status

- Answered: priming is **non-blocking** (failures logged, deploy still succeeds), warms **deployed URLs at the CDN** (not CDA directly, no dynamic rate-limit adjustment), and runs in **controlled batches**.
