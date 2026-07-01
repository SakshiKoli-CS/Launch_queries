QUESTION

Customer **Schumacher Homes** saw a Contentstack **Astro sample-app doc page marked "unmaintained"** and asks, as a planning point (they may migrate from **Next.js → Astro** alongside a content-model refactor): **does Launch still support Astro sites?**

_Keywords: does Launch support Astro, Astro sample app doc unmaintained deprecated, Astro still supported Launch, other frameworks option Launch UI, Astro example repository best practices, migrate Next.js to Astro, framework support despite doc marked unmaintained._

ANSWER

## Yes — Launch fully supports Astro (the "unmaintained" doc label ≠ dropped support)

- **Launch fully supports Astro sites**, with **all existing features and rendering techniques** (SSG/SSR/etc.) — **a documentation page being marked "unmaintained" does NOT mean the framework is unsupported.** The doc label is about that sample-app page's upkeep, not Launch's runtime support.
- **How to use it:** select **"Other frameworks"** in the Launch project setup/interface when configuring the Astro app.
- **Reference:** a comprehensive **Astro example repository** demonstrates Astro + Contentstack integration best practices — useful for evaluating a Next.js → Astro migration.

## Generalized guidance (for future similar queries)

- **"Is <framework> still supported? — the docs page says unmaintained."** → an **unmaintained/deprecated sample-app doc page is not a statement about Launch support.** Launch supports a broad set of frameworks (Next.js, Astro, Nuxt, Gatsby, SvelteKit, 11ty, etc.); frameworks without a first-class preset are configured via **"Other frameworks."**
- **Astro specifically:** **fully supported**, all rendering modes; set up via **"Other frameworks"**; see the **Astro example repo** for best practices.
- Related: [Astro SSG → full rebuild, no selective page builds (use SSR + revalidation)](launch-astro-ssg-full-rebuild-no-selective-page-builds-use-ssr-revalidation.md), [SSG rebuilds / deploy failures → SSR recommendation](launch-uva-frequent-deploy-failures-ssg-rebuilds-network-electron-ssr-recommendation.md), [Edge Functions TS/WinterCG runtime](../edge-functions-rewrites/launch-edge-functions-typescript-support-wintercg-runtime.md).

## Status

- **Answered.** **Astro is fully supported on Launch** (all features/rendering techniques) despite the sample-app doc page being flagged "unmaintained" — configure via **"Other frameworks"** in Launch; pointed the customer to the **Astro example repo** for their Next.js → Astro migration evaluation.
