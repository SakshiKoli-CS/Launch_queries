QUESTION

Prospect **McKinsey** is evaluating Launch's **scalability and compute** for **large-scale SSG (~60k pages)** and runtime delivery. Questions:

- **Compute/build:** How does Launch handle ~60k-page SSG workloads? Are there **configurable compute tiers / autoscaling** during build?
- **Scaling:** Does Launch support **parallelized or incremental builds** for large content volumes? Any **benchmarks / recommended limits** for SSG builds?
- **Runtime:** What **real-time throughput** for page delivery? Is there **global CDN caching + edge delivery** built in, and **SLAs** on response time / availability?

_Keywords: large SSG 60k pages, build time limit 60 minutes, build hours free tier 50, paid 500 GB-Hours, 250 build 250 serverless execution, soft limit add-on, build compute not configurable, parallel builds not exposed, incremental builds cache revalidation Automate, no fixed RPS ceiling, global CDN edge caching, auto SSL, enterprise SLA 99.9% availability, cache priming reduce build time, rendering strategies SSG SSR._

ANSWER

## Build capacity & limits

- **Each deployment can run up to 60 minutes.** Generating **~60k pages typically fits within this** with standard framework optimizations.
- **Build/compute limits:**
  - **Free tier:** **50 build hours/month** (hard limit).
  - **Paid tiers:** **500 GB-Hours/month total** = **250 GB-Hours build** + **250 GB-Hours serverless execution (SSR + Cloud Functions)**. On paid plans this is a **soft limit, increasable as an add-on.**
- **Build compute is NOT user-configurable today**; the 250 GB-Hours build allocation can be **raised as an add-on.** Build performance depends mainly on **application architecture / implementation / build optimizations.**

## Parallel & incremental builds

- **Parallelized builds:** **not exposed as a configurable option** today.
- **Incremental builds:** supported via **cache-control headers + automated CDN cache revalidation (Automation Hub)** — i.e. **incremental-regeneration behavior WITHOUT filesystem-based ISR.** (See Revalidate CDN Cache using Automate.)

## Reducing build time / scaling large SSG

- **Cache Priming:** preloads frequently-accessed pages onto the **CDN cache during deployment** so latest versions are available without an initial request — pages served from the **CDN rather than fully generated as static files at build**, **reducing build time** while keeping static-like runtime performance. Relies on proper **cache-control headers.**
- **Mixed rendering strategies** (common pattern):
  - **SSG** for purely static, rarely-changing pages
  - **SSG + Cache Priming** for high-traffic static pages
  - **SSR + CDN caching** for dynamic / frequently-changing pages
  - **SSR + Cache Priming** to reduce cold starts and avoid building large segments at deploy
  - → keeps **build times short** while delivering high performance globally.

## Runtime delivery, CDN & SLA

- Delivered through a **global CDN** built to scale with **high throughput and sudden spikes.** **No fixed RPS ceiling** — actual throughput depends on the **caching strategy / cache-control** implementation.
- **Customer load testing supported** — request a **controlled load test** to validate specific peak-RPS requirements (must be coordinated/approved).
- Includes: **global edge caching**, **automatic SSL provisioning**, **enterprise SLAs (99.9%+ availability** for delivery services), **full caching control via HTTP cache-control headers.**

## Generalized guidance (for future similar queries)

- **Large SSG (tens of thousands of pages):** feasible within the **60-min/deploy** limit + framework optimization; the lever isn't a bigger build box (**build compute isn't user-configurable**) but **architecture** — **Cache Priming + mixed SSG/SSR rendering** to cut what's generated at build.
- **No parallel-build toggle; no filesystem ISR** — get **incremental** behavior via **cache-control + CDN revalidation (Automate)**.
- **Build quotas:** free **50 build hours (hard)**; paid **500 GB-Hours (250 build / 250 serverless), soft, add-on increasable.**
- **Runtime:** **no fixed RPS cap** (caching-dependent), **global edge CDN + auto SSL**, **99.9%+ delivery SLA**; validate peak RPS via a **coordinated load test**.
- Related: [usage limits: build time / server execution / data transfer](launch-usage-limits-build-time-server-execution-data-transfer.md), [Astro SSG full-rebuild → SSR + revalidation](launch-astro-ssg-full-rebuild-no-selective-page-builds-use-ssr-revalidation.md), [cache priming exact-paths / build-time gen](../caching-cdn/launch-cache-priming-exact-paths-no-wildcard-generate-launchjson-at-build.md), [load testing pre-approval](../logs-monitoring/launch-load-performance-testing-allowed-must-be-preapproved-checklist.md).

## Status

- **Answered (presales reference).** Key facts: **60-min/deploy**, **~60k pages fine**; **build compute not configurable** (250 GB-Hours build, soft/add-on); **no parallel builds**, **incremental via cache-control + Automate revalidation (no filesystem ISR)**; **Cache Priming + mixed SSG/SSR** to scale builds; **no fixed RPS ceiling**, **global CDN + auto SSL + 99.9%+ SLA**; **load testing on request (coordinated).**
