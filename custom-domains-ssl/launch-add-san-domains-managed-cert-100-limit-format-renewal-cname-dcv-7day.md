QUESTION

Customer **PODS** (Org `bltf391054f55c32754`, Azure NA, SF case **00050686**) wants to add **~150 domains to the SAN list** on the Contentstack-managed SSL cert for `pods.com`. All domains are PODS-owned and **redirect to `pods.com`**; they want them on the cert to **avoid HTTPS warnings before the redirect**. They manage redirects themselves — **purely certificate-level support.** Before sharing the list they ask:

1. Can we **extend SAN entries** on the managed cert?
2. Is there a **limit on the number of SAN domains** (technical/policy)?
3. What **format** should they provide the domain list in?
4. Any **renewal / automation / propagation** considerations?

_Keywords: add SAN domains to managed certificate, extend SAN entries, 100 SAN domains per certificate limit, split across multiple certificates, domain list format CSV plain text, automatic renewal SAN entries, Fastly service per domain, CNAME _acme-challenge fastly-validations.com DCV, TLS provisioning 7-day validity window refresh, HTTPS warning before redirect._

ANSWER

## 1. Yes — SAN entries can be added to the managed cert

- Additional **SAN entries can be added** to the existing certificate we manage for `pods.com`.

## 2. Limit — 100 SAN domains per certificate

- The **CDN supports up to 100 SAN domains per certificate.** Beyond 100, **split across multiple certificates** (here ~160 domains → handled via **multiple Fastly services / certs**, 100 + the remainder).

## 3. Domain list format — any clean, deduplicated list

- **Any format works** (CSV, plain text, one domain per line). A **clean, deduplicated** list processes fastest.

## 4. Renewal / automation

- **Renewals are automatic.** All added SAN entries are renewed as part of the **standard automated renewal workflow** — **no extra steps** for the customer.

## Provisioning mechanics (what actually happened)

- Each domain is added to a **Fastly service** (apex redirects run through Fastly — see related entry). For a large batch, a **Fastly domain soft-limit increase** was needed (raised to **160** for the service).
- **TLS provisioning requires the customer to add a DCV CNAME per domain**, e.g.:
  ```
  CNAME  _acme-challenge.<domain>  →  <token>.fastly-validations.com
  ```
- **These DCV validation records have a 7-day validity window.** If the customer doesn't add the CNAMEs before expiry, the records must be **regenerated** (support can **refresh** for another 7 days). **Prompt the customer to add all CNAMEs within the window.**

## Generalized guidance (for future similar queries)

- **"Add many domains to the SAN list of our managed cert (so they don't HTTPS-warn before redirecting)?"** → **Yes**, up to **100 SAN domains per certificate**; **>100 splits across multiple certs/services.** Provide a **clean deduplicated list (any format)**. **Renewals are automatic** for SAN entries.
- **Provisioning is per-domain DCV:** customer adds **`_acme-challenge.<domain>` CNAME → `…fastly-validations.com`**; **validation records expire in 7 days** → add promptly or they're regenerated/refreshed.
- Large batches may need a **Fastly per-service domain soft-limit increase** (internal).
- Related: [apex down / 100-domains-per-service / Fastly = apex redirect (same customer)](launch-apex-down-tls-san-misconfig-100-domain-limit-fastly-apex-redirect.md), [cert pending → DCV CNAME not TXT](launch-cert-pending-dcv-needs-cname-not-txt-domain-limit-golive.md), [add domains / domain-limit increase](launch-add-new-domains-decommission-repurpose-environment.md).

## Status

- **Resolved.** SAN entries **can be added** (**max 100/cert**, split beyond that); **any clean deduplicated list** format; **renewals automatic.** ~160 domains provisioned via **Fastly services** (soft limit raised to 160) with **per-domain `_acme-challenge` CNAME → `fastly-validations.com`** DCV (**7-day validity**, refreshed when expired). Customer **tested and confirmed good**; ticket closed.
