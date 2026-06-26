QUESTION

Customer **WillowTree Inc** (SF case **00052798**) builds their site with **Astro** (a static site generator). Their concern: **the ENTIRE website must be rebuilt on every Launch deploy**, even for **new or edited single pages** — there's no selective/per-page build. They note **Gatsby (Gatsby Cloud / RSG)** could **selectively build/publish individual pages or a list of pages**. Are there plans to support **selective page builds/deploys** on Launch? They tried sync APIs and other ways to selectively publish/deploy pages without success.

_Keywords: Astro static site generator full rebuild, rebuild entire site every deploy, selective page build, incremental build, Gatsby Cloud RSG selective page update, page-level updates without full redeploy, SSR hybrid on-demand rendering, cache priming cache-control, on-demand cache revalidation, Launch framework-agnostic, no RSG equivalent._

ANSWER

## Why the full rebuild happens — Astro is static by default

- Astro builds **fully static** by default, so **fully static builds require a complete site rebuild on every deployment.** There is **no framework-specific selective/incremental page build** on Launch.
- The Gatsby capability they remember is **Gatsby Cloud (RSG)** — a **framework/hosting-specific** feature — **not** the Gatsby framework itself. **Launch is framework-agnostic and does not provide framework-specific functionality like RSG.**

## Recommended approach — SSR / hybrid rendering + on-demand cache revalidation

Instead of selective static builds, achieve **page-level updates without a full redeploy** using standard web mechanisms:

- **Use Astro SSR or hybrid (on-demand) rendering** — pages render **at request time** and **fetch the latest content from Contentstack**, so **content-only changes don't require rebuilding the whole site.**
- **Performance:** **cache-prime** frequently accessed pages to the CDN and set **appropriate `Cache-Control` headers** for fast delivery while still reflecting updates.
- **Cache behavior:** **Launch does NOT auto-purge the CDN cache on content publish.** It supports **on-demand cache revalidation** — trigger invalidation programmatically via **Launch APIs** or **Automation Hub** when content updates. After revalidation, the **next request triggers SSR and re-caches** the page.
- This gives a **similar outcome to RSG's page-level updates** — **on-demand, per-page freshness without a full site redeploy** — and works **across all supported frameworks** (current and future), since it relies on standard caching + revalidation rather than a framework-specific build.

References: Astro SSR / on-demand rendering docs; Launch Astro example repo; Cache Priming; On-demand cache revalidation; Launch APIs; Automation Hub.

## Generalized guidance (for future similar queries)

- **"Can Launch selectively build/deploy individual pages (like Gatsby Cloud/RSG)?"** → **No** — Launch is **framework-agnostic**, no RSG-style selective static builds; **fully static (Astro/SSG) needs a full rebuild per deploy.**
- **The platform-native equivalent** is **SSR/hybrid rendering + on-demand cache revalidation**: render pages at request time, **cache-prime + `Cache-Control`** for speed, and **purge/revalidate per-page via Launch APIs / Automation Hub** on publish (Launch does **not** auto-purge on publish). Net effect ≈ page-level updates without a full redeploy.
- This is the same architectural recommendation given for **large SSG sites with slow/fragile full rebuilds** — see [UVA SSG→SSR recommendation](launch-uva-frequent-deploy-failures-ssg-rebuilds-network-electron-ssr-recommendation.md).
- Related: [cache priming exact-paths / build-time generation](../caching-cdn/launch-cache-priming-exact-paths-no-wildcard-generate-launchjson-at-build.md), [Revalidate CDN Cache](../caching-cdn/launch-revalidate-cdn-cache-one-parameter-per-request.md), [stale published content / Cache-Control](../caching-cdn/launch-stale-published-content-cdn-cache-no-cache-control-headers-next.md).

## Status

- **Answered.** **No selective/incremental static page builds** (RSG is Gatsby-Cloud-specific; Launch is framework-agnostic). Recommended **Astro SSR/hybrid + cache priming + on-demand cache revalidation (Launch APIs / Automation Hub)** to get **per-page updates without full redeploys**. Customer **acknowledged and asked to close** the ticket.
