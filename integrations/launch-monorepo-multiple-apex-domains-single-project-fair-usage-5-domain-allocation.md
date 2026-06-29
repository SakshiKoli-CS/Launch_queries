QUESTION

Can a customer use a **monorepo for multiple brands** and configure **each brand's apex domain as a custom domain within a single Launch project** — or must they **buy one Launch project per apex domain**?

_Keywords: monorepo multiple brands single project, multiple apex domains one project environment, one project per apex domain required, custom domains multiple apex, fair usage server compute per project, 5 domains standard allocation, additional cost beyond 5 domains, configure apex subdomains self-serve entitlement._

ANSWER

## Yes — multiple apex/subdomains in a single project/environment is supported

- You **can add multiple apex or subdomains to a single environment/project** in Launch — you do **NOT** need one project per apex domain. **Several customers already do this** for multi-brand setups.
- This fits the case of **one project + one shared code release process across all brands** (the monorepo deploys once, serving multiple branded apex domains).

## Constraints — fair usage of compute + the 5-domain allocation

- **No technical limitation** — the only practical limits are:
  - **Fair usage of server compute per project** — multiple sites on one project share that project's compute; **low-to-moderate-traffic** brands comfortably fit within fair usage.
  - **Standard allocation is 5 domains (apex or subdomains) per project** — **additional cost applies beyond 5.**

## Self-serve apex configuration

- With the **new custom domain feature**, users can **configure any combination of apex domains + subdomains themselves**, as long as they're **within the entitlements set in super admin** (previously this needed Launch-team involvement). *(That feature hadn't released yet at the time of the thread.)*

## Generalized guidance (for future similar queries)

- **"One project per apex domain, or many apex domains in one project?"** → **Many apex/subdomains can live in one project/environment** — no per-apex project requirement. Right fit for a **monorepo serving multiple brands with a shared release process.**
- **Watch two things:** **fair-usage server compute** (all brands share the project's compute — fine for low/moderate traffic) and the **5-domain standard allocation** (more domains = additional cost; raise the entitlement).
- High-traffic brands or independent release cadences may still justify **separate projects** (compute isolation), but it's a **business/architecture choice, not a technical requirement.**
- Related: [add domains / domain-limit increase](../custom-domains-ssl/launch-add-new-domains-decommission-repurpose-environment.md), [domain limit is global across environments](../edge-functions-rewrites/launch-domain-migration-redirect-old-domains-to-subfolders-edge-function-domain-limit.md), [100 SAN domains per cert](../custom-domains-ssl/launch-add-san-domains-managed-cert-100-limit-format-renewal-cname-dcv-7day.md).

## Status

- **Answered.** A **monorepo with multiple brands can use a single Launch project with multiple apex/custom domains** — **no per-apex project requirement** (many customers do this). Limits: **fair-usage compute per project** + **5-domain standard allocation** (more = added cost). New custom-domain feature lets users **self-configure apex/subdomains within entitlements**.
