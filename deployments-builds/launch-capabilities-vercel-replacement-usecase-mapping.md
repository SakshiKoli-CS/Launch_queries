QUESTION

Customer **Megaworld** is evaluating **Launch as a replacement for Vercel** and asked for a mapping of their use cases to Launch capabilities + setup considerations. Their evaluation areas: frontend hosting/deploy, backend/API layer, database integration (Neon/PostgreSQL), CMS integration (HubSpot), build/deploy automation (incl. cron), env vars/secrets, performance/scalability, developer experience (deploy/rollback, logging), and database observability.

_Keywords: Launch vs Vercel, Vercel replacement, capabilities mapping, use case fit, evaluation, Cloud Functions, Edge Functions, runtime limits, 30s timeout, 1024MB memory, read-only filesystem, Node versions, Neon Postgres external DB, HubSpot CMS, deploy hooks, cron jobs, environment variables, rollback, log targets, Launch Analytics._

ANSWER

Launch supports the **majority** of these use cases. Mapping with key considerations:

## 1. Frontend hosting & deployment
- Supports modern frameworks (**React, Next.js**); **preview / staging / prod environments** for validation before promotion. (Framework support + Environments docs.)

## 2. Backend / API layer
- **Cloud Functions** + **native framework API routes** (e.g. Next.js API routes) for server-side logic and orchestration between frontend, CMS, and DB. **Edge Functions** for low-latency ops (auth/personalization).
- **Important:** Node backends run **on-demand per request — NOT a continuously persistent server.**
- **Runtime limits:** **30s execution timeout/request**, **1024 MB memory**, **Node 20.x/22.x/24.x**, **read-only filesystem except `/tmp`**.

## 3. Database integration (Neon/PostgreSQL)
- **No managed database service** — integrate **external** DBs (Neon/Postgres). Secure connections via **environment variables**; CRUD via backend functions. **Database branching stays managed in Neon.**

## 4. CMS integration (HubSpot)
- Fully supported via standard **headless** patterns — fetch via HubSpot APIs; **webhooks trigger rebuilds / cache revalidation**; clean content/presentation separation.

## 5. Build & deployment automation
- **Git-based deployments**, **CMS-triggered rebuilds via webhooks**, **deploy hooks** for automation. (Automation Hub + Deploy Hooks docs.)
- **Cron jobs:** the customer uses **Vercel Cron** today — confirm the equivalent on Launch before committing (e.g. scheduled triggers via Automate / external scheduler hitting a deploy hook or API route). *Treat cron parity as a check item; not explicitly confirmed in-thread.*

## 6. Environment variables & secrets
- **Environment-specific configs** (dev/staging/prod), secure storage for API keys/DB URLs/tokens, environment isolation.

## 7. Performance & scalability
- Globally distributed **CDN**, **Edge Functions**, **Cache-Control–based caching**, **cache priming**; static + dynamic optimization.

## 8. Developer experience
- **Git-based deploy** with promotion across environments. **Rollback was in progress** (coming weeks at the time) — verify current availability.
- **Centralized build + runtime logs**; **Log Targets** route logs to external observability. App-level visibility into deployments/builds/runtime (external systems monitored via their own tools).

## 9. Database management & observability
- **Not provided by Launch** — provision/manage/query/monitor the DB **externally** (e.g. Neon Console). Launch focuses on **application-level observability** (deployments, runtime, logs); stream app metrics to external **Log Targets**; **Launch Analytics** for usage/performance insights.

## Key gotchas for a Vercel migrant (summary)

- **No persistent server** — request-scoped execution; **30s/1024MB/read-only-FS-except-`/tmp`** limits (see [Node-24 / read-only FS entry](launch-node24-upgrade-chromium-pdf-readonly-fs-pin-node22.md)).
- **No managed DB** — bring your own (Neon/Postgres) via env vars; branching/observability stay in the DB provider.
- **Cron** — validate the scheduling story vs Vercel Cron before relying on it.
- **Rollback** — confirm GA status.

## Generalized guidance (for future similar queries)

- **"Can Launch replace Vercel for our stack?"** → Yes for **frontend hosting (React/Next), API routes/Cloud Functions, Edge Functions, env-scoped secrets, Git + webhook + deploy-hook automation, CDN caching/cache-priming, centralized logs/Log Targets/Analytics**.
- **Flag the differences up front:** request-scoped (non-persistent) execution with **30s/1024MB/read-only-FS** limits; **no managed database** (external + env vars); **cron parity** to be confirmed; **rollback** availability to confirm.
- For DB/observability, set expectations that **Launch is app-layer**; database hosting/monitoring lives with the external provider.

## Status

- Provided the full use-case → capability mapping with docs links; offered a discovery call. Customer (partner) found it helpful and shared it with Megaworld.
