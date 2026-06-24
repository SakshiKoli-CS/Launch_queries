QUESTION

Customer **BMW** (scale: **100+ markets × thousands of pages**) asked about **ISR on Launch** and the **cache-revalidation cap**:
- Is **native ISR** on the roadmap and when?
- The docs show a cap of **2,000 on-demand cache revalidations/day at Enterprise** — is it **negotiable**?
- What are the **limitations of cache revalidation via HTTP headers + Automate cache flushing vs Vercel's native ISR**?

_Keywords: ISR support, Next.js ISR App Router, revalidatePath revalidateTag, Pages Router ISR, 2000 cache revalidation limit, revalidation cap negotiable, SSR cache headers, stale-while-revalidate, cache tags purge, Automate revalidation, deploy hooks, Vercel ISR comparison._

ANSWER

## Native ISR support

- **Next.js native ISR (`revalidatePath` / `revalidateTag`) is NOT supported for App Router** on Launch.
- **ISR IS supported for Pages Router** (Next.js **12.2+**).
- For **App Router**, the recommended approach is **SSR + Cache-Control headers** (ISR-like, platform-independent).

## ISR-like behavior on Launch (App Router)

1. **SSR + `Cache-Control` headers** — set CDN caching explicitly (e.g. **`s-maxage` + `stale-while-revalidate`**) so the CDN serves cached content and **refreshes in the background**.
2. **On-demand CDN cache revalidation** — via **Automate** and the **Revalidate CDN Cache** action / **REST API**. This is **CDN cache revalidation**, **not** native Next.js ISR primitives (`res.revalidate()`, `revalidatePath`, `revalidateTag`).
   - **Correction:** on-demand revalidation is **NOT available via deploy hooks** (a common misconception).
3. **Cache tags** — purge **multiple URLs with a single API call**, reducing the number of individual revalidation requests at scale.

## The 2,000/day revalidation cap

- **The 2,000 limit CAN be increased** to a reasonable number based on the customer's requirements (i.e. **negotiable at Enterprise**).
- **It doesn't map 1:1 to pages** — you can purge by **paths, cache tags, or domains**, so a **single request can invalidate many pages**. With cache tags/domain purges, 2,000 requests cover far more than 2,000 pages.
- **Every deployment/redeployment automatically purges the existing cache** (so full refreshes on deploy don't consume the on-demand quota).
- Ref: Revalidate CDN Cache docs; Platform Limits (cache revalidation by plan).

## vs Vercel native ISR

- Launch gives **framework-agnostic, CDN-layer** caching/revalidation (`Cache-Control` + cache tags + Revalidate API/Automate) rather than Next.js ISR primitives. For **App Router**, that's the equivalent path; for **Pages Router**, native ISR works (12.2+).

## Generalized guidance (for future similar queries)

- **"Does Launch support ISR?"** → **Pages Router: yes (12.2+)**; **App Router: no native ISR** — use **SSR + `s-maxage`/`stale-while-revalidate`** + **Revalidate CDN Cache** (Automate/REST) + **cache tags**.
- **"Is the 2,000/day revalidation cap a blocker at scale?"** → The cap is **negotiable/increasable**, and it's **per revalidation request, not per page** — **cache tags / path / domain** purges invalidate many pages per request. Deploys auto-purge for free.
- **No on-demand revalidation via deploy hooks** — use **Automate** or the **Revalidate CDN Cache API**.
- **Big multi-market/many-page sites** → lean on **cache tags** to keep request counts well under the cap.

## Status

- Clarified ISR support (Pages yes / App Router no-native), the ISR-like SSR+headers+Revalidate approach, and that the **2,000/day cap is increasable and not page-based** (cache-tag/path/domain purges + auto-purge on deploy).
