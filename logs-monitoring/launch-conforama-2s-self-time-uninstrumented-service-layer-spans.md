QUESTION

Customer **Conforama** (SF case **00056923**, JIRA **CL-3533**, Apikey `bltd50d0089a2c99486`, Project `68b86130f7d015d611211b65`) reported via **Sentry** that several requests on **Launch** exceed **2 seconds**, while the **same code on Vercel doesn't reproduce** it. Not URL-specific — occurs on **PDP and PLP** pages (`*.eu-contentstackapps.com`). They asked for the reasons for the latency, known limits, and any optimization needed.

_Keywords: Sentry 2s self-time, resolve page components span, uninstrumented service layer, missing custom spans, http.client spans, latency Launch vs Vercel, fetchEmerchproductlist brandPageService getFeatureflag, sequential async, cache handling, custom spans instrumentation._

ANSWER

## Finding — the ~2s is uninstrumented in-app work (service/domain layer)

After reviewing the Sentry trace and the (shared) codebase:
- The parent span shows **~2 seconds of self-time** on **"resolve page components"** — the **unaccounted portion of the waterfall**. In Sentry, that **self-time = work happening within the application not captured by child spans** (missing instrumentation).
- **Visible vs not:** the **`http.client` spans** (auto-instrumented outgoing HTTP calls) are visible; the **remaining ~2s gap is time in async operations in the domain/service layer that aren't instrumented.**

## Where the time is likely spent (from the code flow)

The brand page runs **multiple operations**, several not visible in the trace:
- `fetchEmerchproductlist()`
- `fetchSegmentEntryByTitle()`
- `brandPageService()`
- `getAdBannerConfig()`
- `getVariableConfiguration('ID_GAM')`
- `getFeatureflag()` — **invoked twice**

Some external API calls show up, but **internal processing — business logic, cache handling, and sequential execution — is not instrumented**, so Sentry **aggregates that time into the parent span's self-time**, making it impossible to attribute the latency to a specific function.

## Recommendation — add custom spans to localize the bottleneck

- Wrap **domain-level functions, service/DB/cache calls, and sequential async flows** in **custom Sentry spans**. This **breaks the aggregated self-time into granular, traceable operations** so the real hotspot (e.g. a slow/sequential call, a cache miss, the double `getFeatureflag()`) becomes visible.
- Likely culprits to look at: **sequential (not parallelized) async calls**, **cache handling**, and **duplicate calls** (e.g. `getFeatureflag()` twice).

## To compare fairly with Vercel (requested data)

- Share **Next.js & Node versions**, and any **Launch-vs-Vercel config/optimization differences**.
- Run a **latency check on Vercel** (repeatedly hit the same PDP/PLP), capture **HAR/network logs**, and note whether latency is **consistent or intermittent** — this helps separate **caching / request spikes / cold starts** from app processing.

## Generalized guidance (for future similar queries)

- **"Requests >2s on Launch (Sentry), but fine on Vercel"** → a large **span self-time** is **uninstrumented in-app work** (domain/service layer), **not** platform latency. **Add custom spans** around service/cache/sequential-async code to localize it.
- **Auto-instrumented `http.client` spans only cover outbound HTTP** — internal business logic/cache/sequential awaits are invisible until you wrap them.
- **Look for sequential awaits, cache misses, and duplicate calls** (e.g. a function invoked twice) as common 2s-class culprits.
- Compare platforms with **HAR + repeated hits** and **matched Next/Node versions + config** before attributing to Launch.
- Related (higher-level / geographic framing of the same case): [launch-latency-investigation-vs-vercel-sentry-self-time-instrumentation.md](launch-latency-investigation-vs-vercel-sentry-self-time-instrumentation.md).

## Status

- Findings shared: the **~2s is uninstrumented service-layer self-time** (not Launch infra); recommended **custom spans** + Vercel HAR comparison + version/config details. Tracked **CL-3533**; awaiting customer (held open per the standard window).
