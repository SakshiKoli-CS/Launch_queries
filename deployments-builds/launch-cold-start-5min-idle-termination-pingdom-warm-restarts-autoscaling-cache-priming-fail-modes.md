QUESTION

Customer **Highstreet Insurance Partners** (via HUGE, SF case **00050755**; sites `www.myhighstreet.com`, `www.highstreetpartners.com` + `*.azcontentstackapps.com`; monorepo with **two Next.js apps**) reports, across all environments:

- **Intermittent server restarts** → **slow first load (~30–40s)** (cold start); fine once warm. Affects both customer- and partner-facing sites.
- Follow-ups: **how long do servers stay up with no traffic? cold starts seem frequent (not just after deploy).** After launch: **"servers restarting 98 times / 35 times in 24h, sometimes within 2 minutes."**
- **Cache priming partially effective** — same config, but **one site primed successfully and the other failed.** Why?
- How is an environment **marked as a production site** (for the health check), and **what request** is made?

_Keywords: cold start 30-40s first load, server restarts frequent, Azure container idle 5 minutes terminated, Pingdom health check keep warm, mark production site manual go-live, root domain GET every minute, restarts = autoscaling up down not failures, cache priming failed one site not other, missing restrictive cache-control headers, primed URL returns error, priming failures don't block deploy, s-maxage._

ANSWER

## Cold start cause — idle Azure containers terminate after 5 minutes

- On Launch (**Azure**), **any container idle for 5 minutes is automatically terminated** → the next request **cold-starts** a new container (the ~30–40s first load). This is **expected in non-production** envs where traffic is intermittent (dev/test cold-start often).
- **Production fix — Pingdom health checks** keep **at least one instance warm at all times** (also uptime monitoring), so cold starts stop being an issue once live.

## How a site is marked "production" + what request runs

- **Production environments are marked manually by the Launch team as part of go-live** (when the domains are added on Launch, the team adds the **Pingdom checks**).
- The **Pingdom check hits the root domain every minute with a simple GET** (e.g. `http://myhighstreet.com/`, `http://highstreetpartners.com/`).

## "98 / 35 restarts in 24h, some within 2 minutes" — that's autoscaling, not failures

- **Frequent "restarts" are Launch scaling instances up/down based on traffic**, not crashes. Pingdom guarantees **≥1 container stays running** even with no traffic; the rest scale with load — which shows as many instance start/stops. **Not an error condition.**

## Why cache priming succeeds on one site, fails on the other — two failure modes

Cache Priming **fetches the page response at the edge/CDN**; whether it caches depends on the response. It can fail to cache when:
1. **`Cache-Control` headers missing or restrictive** — Launch still fetches the response, but **won't cache it** if the headers prevent/don't allow caching. (Two sites with the "same config" can differ if their actual page responses emit different/again default headers.)
2. **The primed URL returns an error / invalid response** — priming for that specific page won't succeed; **visible in application logs** with server-side logging.
- **Priming failures don't block or fail the deployment** — it continues for the other URLs and the deploy still goes live. The UI shows the **count of URLs successfully primed.**
- Recommendation: add **`Cache-Control` with `s-maxage`** on pages to get reliable CDN-level caching (don't rely on Next.js default headers).

## Generalized guidance (for future similar queries)

- **Cold start / slow (~30–40s) first load + "frequent restarts"** → idle **Azure containers terminate after 5 min**; first request cold-starts. **Production stays warm via Pingdom** (≥1 instance, GET root domain every minute) — **marked manually by Launch at go-live**. **"Many restarts" is autoscaling up/down, not crashes.**
- **Cache priming works on one site but not another (same config)** → check the **actual page response headers** (missing/restrictive `Cache-Control` won't cache; **Next.js default headers** may not) and **whether the URL errors** (logged). Priming **fetches but only caches with proper headers**; failures **don't fail the deploy**. Use **`s-maxage`**.
- Related: [cold start first-load / cache priming warms / go-live (same case)](launch-cold-start-first-load-45s-idle-container-cache-priming-warms-golive.md), [idle environment Warm-Up Screen](../logs-monitoring/launch-idle-environment-sleep-error-warm-up-screen.md), [cold start + health-check warming at go-live](launch-npm-err-invalid-arg-type-path-null-security-fix-revert-sso-ssr-cache-cold-start.md), [cache priming exact-paths / build-time gen](../caching-cdn/launch-cache-priming-exact-paths-no-wildcard-generate-launchjson-at-build.md).

## Status

- **Explained.** Cold starts = **Azure containers idle >5 min are terminated**; **Pingdom health checks** (manual at go-live, **GET root domain/min**) keep ≥1 warm in production. **"98/35 restarts/day" = autoscaling up/down, not failures.** **Cache priming** caches only with proper **`Cache-Control` (use `s-maxage`, not Next.js defaults)** and a **valid (non-error) response**; failures are **logged** and **don't block deploys**.
