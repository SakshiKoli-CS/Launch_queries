QUESTION

Customer **UVA Health** (Org `bltdcc012133cba1c83`, Azure NA, SF case **00051032**) is **migrating several domains/subdomains to Contentstack** and wants each old domain to **redirect to specific subfolders of `www.uvahealth.com`**. Domains:

- `blog.uvahealth.com`, `newsroom.uvahealth.com`, `uvaphysicianresource.com`, `makingofmedicine.virginia.edu`

Examples (note **path transformation**, not 1:1):
- `https://blog.uvahealth.com/2025/10/17/a-new-treatment-…/` → `www.uvahealth.com/healthy-balance/a-new-treatment-…`
- `https://makingofmedicine.virginia.edu/2025/10/07/a-major-manning-milestone/` → `www.uvahealth.com/making-of-medicine/a-major-manning-milestone`

Can this be done in Contentstack by pointing the domains to Launch? (They later hit **domain-limit** errors while adding the domains.)

_Keywords: migrate domains redirect to new URL structure, old domain to subfolder, domain plus dynamic path slug date folder, Edge Function 301 302 redirect, point domains to Launch first, custom domain setup, apex/domain limit 2 increase, global domain limit across environments not per-env, wait till deployment completed add domain disabled, CNAME TXT coordination third party._

ANSWER

## Yes — domain-level redirects on migration are fully achievable with Launch

- Once you **point the domains/subdomains to Launch**, you can implement redirects so any request to the **old sites is routed to the corresponding paths on the new site** (`www.uvahealth.com`). Applies to all four domains.

## Prerequisite — add the domains in Launch FIRST

- **Domains/subdomains must be added to the Launch environment before redirects can work** — only then does Launch receive the requests and apply the redirect logic. (Custom Domain Setup docs.)

## Implementation — Edge Function (because the redirects transform the path)

- The redirect patterns depend on **domain + dynamic path** (article slugs, date-based folders → mapped subfolders), so a **Launch Edge Function** is the recommended approach: it can **detect the incoming domain, parse the URL, and dynamically build the new destination path**, issuing **301/302** redirects.
- This suits consistently-structured blog/archive URLs needing programmatic mapping (e.g. `/2025/10/17/<slug>/` → `/healthy-balance/<slug>`).
- Refs: Edge Functions docs; example repo **`contentstack-launch-examples/launch-edge-function-redirect`**.
- (For redirect *rules* managed in the CMS rather than coded mappings, see the [CMS-driven redirects entry](launch-cms-driven-redirects-no-redeploy-no-per-request-api-edge-api-route-cache.md).)

## Domain-limit mechanics (hit during this migration)

- Adding domains can hit a **domain/apex limit** (started at **2** here: `azuvahealth.azcontentstackapps.com` + `uvahealth.com`). The limit is set on the **plan / Super Admin** and is **increased by Contentstack** on request (needs internal approval).
- **The domain limit is a GLOBAL cap across ALL Launch environments combined — NOT per-environment.** When sizing, **sum the domains across every env** (e.g. UVA: main 6 + childrens 2 + SOM 4 + redirects 3–4 = **~16–17 total** → limit raised to **17**). There is **no per-environment domain limit** to configure.
- Also seen: **"add domain" disabled / 'wait till deployment completes'** even with no deployment in progress — a transient UI/state condition; retrying after a moment worked.
- Plan note: project count is separate (they had 2 projects purchased; raised to 3).

## Generalized guidance (for future similar queries)

- **"Migrate old domains to redirect into subfolders of a new site?"** → **Yes** — **point domains to Launch, add them in Launch first**, then use an **Edge Function** to map **domain + path → new subfolder path** with **301/302** (use the `launch-edge-function-redirect` example). Best when paths are **transformed**, not 1:1.
- **Domain limit is global across all environments, not per-env** — total up every environment's needs and request a single increase; the cap lives on the plan (Contentstack raises it, with approval).
- **"Add domain" disabled / 'deployment in progress' with none running** → transient; retry shortly.
- Related: [add domains / domain-limit increase](../custom-domains-ssl/launch-add-new-domains-decommission-repurpose-environment.md), [apex/100-domains-per-service](../custom-domains-ssl/launch-apex-down-tls-san-misconfig-100-domain-limit-fastly-apex-redirect.md), [CMS-driven edge redirects](launch-cms-driven-redirects-no-redeploy-no-per-request-api-edge-api-route-cache.md), [custom 404 edge function](launch-custom-404-static-nuxt-ssg-edge-function.md).

## Status

- **Answered + unblocked.** Domain-migration redirects (old domain → new subfolders) are **achievable**: **add domains in Launch first**, then an **Edge Function** to map domain+path → new path (301/302; `launch-edge-function-redirect` example). Domain-limit errors resolved by **raising the global limit** (2 → 4 → 6 → ultimately **17 across all envs**) — the limit is **global, not per-environment**; transient "wait for deployment" add-domain block cleared on retry.
