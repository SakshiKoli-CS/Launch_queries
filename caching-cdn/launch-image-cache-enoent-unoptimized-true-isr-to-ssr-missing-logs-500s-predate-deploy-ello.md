QUESTION

Customer **Ello Group / tastecard** (SF case **00045480**, JIRA **CL-2026**, NX monorepo, Next.js **App Router**, EU) hit a cluster of production issues:
1. **Server crash / 500s** with `Failed to write image to cache … Error: ENOENT: no such file or directory, mkdir '/var/task/sourcecode/apps/tastecard/.next/cache'`.
2. After trying cache-dir workarounds, **server logs stopped appearing entirely** (nothing loads as they click around).
3. **Slow production deploys** (~25–27s page loads, deploy page freezes), and **some `/subscribe/*` pages throw 500s**.
The customer felt the guidance was pushing a **full re-architecture** of a setup that "worked until a couple of weeks ago" — treating it as a **business-critical Launch incident**.

_Keywords: Failed to write image to cache ENOENT mkdir .next/cache, /var/task/sourcecode read-only filesystem, image cache write fails, images unoptimized true next.config.js, NX monorepo cacheDirectory NEXT_CACHE_DIR /tmp, ISR not supported on Launch use SSR getServerSideProps cache-control, App Router server components no-store, 500 errors predate V2 deployment, no server logs after deploy, slow deployment 25 seconds, app issue treated as Launch issue._

ANSWER

## Issue 1 — Image-cache ENOENT → set `images.unoptimized: true` (don't move the cache dir)

- **`Failed to write image to cache … ENOENT … mkdir '/var/task/…/.next/cache'`** = Next.js is trying to **write optimized images to a cache dir on a read-only/ephemeral serverless filesystem** (`/var/task/...`).
- **Recommended fix: `images: { unoptimized: true }` in `next.config.js`** (alongside the existing `remotePatterns`). This **stops Next.js from doing on-the-fly image optimization** (which is what writes to `.next/cache`) → no ENOENT.
- **Do NOT** move the cache to `/tmp` or set a `NEXT_CACHE_DIR` env var — those workarounds were **retracted** (the customer's NX-monorepo variant, `cacheDirectory: "{projectName}/tmp/.next/cache"` and `NEXT_CACHE_DIR: '/tmp/.next/cache'`, **didn't fix it and added confusion**). **Revert them.**
- If they want **optimized** Contentstack CMS images, use the **Image Delivery API** (server-side optimization) instead of Next's built-in optimizer.

## Issue 2 — ISR is the wrong model on Launch → use SSR + Cache-Control (+ Automation Hub revalidation)

- The app relied on **Incremental Static Regeneration (ISR)**, which **doesn't behave well on Launch's serverless infra** (same ephemeral-filesystem class of problem — ISR wants to write/regenerate static output on disk).
- **Recommended model on Launch:**
  - **SSR** for pages (Pages Router: `getStaticProps` → `getServerSideProps`; **App Router: server components, force dynamic / `cache: 'no-store'` on fetches** you don't want cached),
  - **Cache-Control response headers** to manage CDN caching,
  - **Automation Hub** for controlled revalidation/content refresh.
- Docs: **Next.js on Launch → App Router cache revalidation**, **Cache-revalidation with ISR**.
- ⚠️ **Framing matters:** present this as **aligning caching with Launch's model**, NOT "re-architect everything." For an App Router app it's largely **cache-directive changes** (`no-store`/dynamic + headers), not a rewrite.

## Issue 3 — "No server logs after deploy" + 500s that PREDATE the V2 deploy

- **Key finding:** the **500 errors on `/subscribe/*` began during the deploy of `release/v1.0.9` (deployment #44908) — ~15 min BEFORE the first V2 deploy** — contradicting the assumption that "V2 broke it." Some subscribe entries worked, others 500'd → **entry/data-specific app errors**, not a blanket Launch outage.
- The **ENOENT image errors had existed for a long time**, well before the recent incident — i.e. **not the trigger** for the new breakage.
- Much of this is an **application issue surfacing on Launch**, not a Launch platform fault. Confirmed by: no telemetry/logging-service problems found on Launch; the page eventually loads (slow, ~25–27s — Launch investigating that separately as a real perf bug).
- **Diagnosis asks that generalize:** get the **environment UID**; **deploy the same (V2) code to a non-prod env with PRODUCTION env vars** to reproduce; ask **whether any backend service (CMS, Algolia, Stripe, Auth0, EB_API) changed** around when it "self-resolved" for V1; request **read-only GitHub app access** to inspect the code/config.

## Generalized guidance (for future similar queries)

- **`Failed to write image to cache … ENOENT … .next/cache`** → **`images: { unoptimized: true }`** in `next.config.js` (serverless FS is read-only/ephemeral; Next's image optimizer can't write there). **Don't** chase `/tmp` cache-dir or `NEXT_CACHE_DIR` hacks. Optimize CMS images via the **Image Delivery API** instead.
- **ISR on Launch** → prefer **SSR + Cache-Control headers + Automation Hub revalidation**. App Router: **server components with `cache: 'no-store'`/dynamic** + response headers — frame as a **caching-config change, not a re-architecture**.
- **"Suddenly broke, no logs, some pages 500"** → **correlate the 500 onset against the exact deployment in the Launch UI** (a traffic/500 chart often shows they **predate** the release the customer blames). **Reproduce on a non-prod env with prod env vars**, check for **backend-service changes**, and separate **long-standing errors** (old ENOENT) from the **new** trigger.
- Related: [ISR support + revalidation cap (roadmap/limits)](launch-isr-support-cache-revalidation-limit-cap.md), [App Router returns RSC payload behind CDN](launch-nextjs-app-router-rsc-payload-instead-of-html-cdn-rsc-affected-paths.md), [stale published content / no Cache-Control](launch-stale-published-content-cdn-cache-no-cache-control-headers-next.md), [user data leak: `no-cache` insufficient → `no-store`](launch-user-data-leak-between-sessions-no-cache-insufficient-use-no-store.md), [CF001 on cache miss = app crash/unhandled error](../logs-monitoring/launch-cf001-on-cache-miss-app-crash-unhandled-error-restart-mechanism.md), [Cloud Functions not supported in monorepos (NX/monorepo caveats)](../cloud-functions/launch-cloud-functions-monorepo.md).

## Status

- **In progress → app-issue on Launch (CL-2026).** **Image ENOENT** fix = **`images: { unoptimized: true }`** (retract the `/tmp`/`NEXT_CACHE_DIR` workarounds; use **Image Delivery API** for optimized CMS images). **ISR** → migrate to **SSR + Cache-Control + Automation Hub** (framed as caching alignment, not re-architecture). **Missing-logs/500 incident:** 500s on `/subscribe/*` **predated the V2 deploy** (started in deployment #44908 / `v1.0.9`), long-standing ENOENT was **not** the trigger → largely an **application/data issue surfacing on Launch**; Launch separately investigating the **~25–27s slow page load** as a real perf bug. Repro plan: same code on **non-prod env + prod env vars**, share **env UID**, check backend-service changes, **read-only GitHub access** granted for inspection.
