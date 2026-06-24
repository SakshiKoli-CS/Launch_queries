QUESTION

Customer **Conforama** (JIRA **CL-3533**) raised **latency/performance** concerns about their Launch-hosted app (and how it compares to Vercel). A **Sentry trace** showed **~2 seconds of self-time** on the **"resolve page components"** span. Is the latency a Launch hosting issue?

_Keywords: latency investigation, Launch vs Vercel performance, slow page, Sentry trace self-time, resolve page components 2s, missing instrumentation, unaccounted span time, k6 load test, region Ireland Paris, app-side latency._

ANSWER

## Launch vs Vercel comparison — difference is mostly geographic

- The team deployed the app on **Vercel (Paris)** and **Launch (Ireland)** and ran **k6 load tests (2–10 virtual users)** from an **EC2 instance in Paris**.
- Because the **Launch instance is in Ireland** and **Vercel in Paris**, a **few-milliseconds** latency difference is expected purely from **geographic distance** — not a platform performance gap. (Two k6 reports shared for reference.)

## Sentry trace — the ~2s is in-app work (missing instrumentation), not Launch

- The parent span showed **~2 seconds of self-time** (the unaccounted portion of the waterfall) on **"resolve page components."**
- In Sentry, **self-time** like this typically represents **work happening within the application that isn't captured by child spans** — i.e. **missing instrumentation**, not Launch infrastructure latency.
- **Interpretation:** the ~2s is being spent **inside the app's page-component resolution**, but without finer-grained spans you can't see *where*. The latency is **app-side**, not platform.

## Recommendation

- **Add child-span instrumentation** around the "resolve page components" logic (and its sub-steps / data fetches) so the ~2s self-time is broken down and the real hotspot becomes visible.
- Compare platforms **fairly** — same client origin and account for **region** (Ireland vs Paris) before attributing latency to the platform.

## Generalized guidance (for future similar queries)

- **"Launch is slower than Vercel"** → compare from the **same origin** and account for **region/geography** (a few ms is just distance, e.g. Ireland vs Paris). Use a consistent **load test (k6)** on both.
- **A large "self-time" on a span (e.g. ~2s on 'resolve page components')** → that's **unaccounted in-app work due to missing instrumentation**, not platform latency. **Add child spans** to localize it; the time is inside the application.
- Don't attribute span self-time to Launch — instrument deeper first.

## Status

- Shared **k6 reports** (Launch vs Vercel) showing the difference is largely **geographic**, and explained the **~2s Sentry self-time** as **app-side, missing-instrumentation** work (not Launch). Awaiting customer response; ticket held open per the standard 10–15 business-day window before closing.
