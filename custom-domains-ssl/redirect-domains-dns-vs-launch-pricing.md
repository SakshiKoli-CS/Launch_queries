QUESTION

**Endress+Hauser** has country-code domains (e.g. `endress.de`) that redirect to subdomains on their main site (e.g. `www.de.endress.com`). The question was whether redirect-only domains need to be configured and purchased in Launch, or whether DNS-level redirects are an option — and what the pricing implication is for each approach.

ANSWER

**Two approaches are available, with different domain-slot costs.**

**Approach 1 — DNS/CDN redirect (redirect-only domain stays outside Launch)**

The redirect (`endress.de` → `www.de.endress.com`) is configured at the customer's DNS provider or CDN. Only the destination domain (`www.de.endress.com`) needs to be added to Launch.

- **Domain slots used: 1** (the destination only)
- The source domain (`endress.de`) is not in Launch and does not consume a slot
- The destination subdomain (`www.de.endress.com`) counts as a **custom domain** — not free even though it is a subdomain

**Approach 2 — Redirect configured in Launch (both domains in Launch)**

The redirect-only domain (`endress.de`) is added to Launch as a custom domain and configured using Launch's **Custom Redirection** feature to redirect to the destination.

- **Domain slots used: 2** (source + destination)
- Both domains count against the customer's domain entitlement
- Reference: [Adding apex domains with redirects](https://www.contentstack.com/docs/developers/launch/custom-domain#adding-apex-domains-with-redirects)

**Cost comparison**

| | Approach 1 | Approach 2 |
|---|---|---|
| Redirect managed by | DNS provider / CDN | Launch Custom Redirection |
| Domain slots consumed | 1 (destination only) | 2 (source + destination) |
| Best when | Customer's DNS supports redirects | Centralised management in Launch preferred |

**Note on subdomains**

Even though `www.de.endress.com` is a subdomain, it still counts as a **custom domain** in Launch and must be purchased/allocated — it is not included for free.

**Exception process**

If the decision to use DNS redirects is purely driven by domain slot limits or cost, an exception can be requested internally to allocate additional domains to the customer at no extra cost, keeping both domains managed within Launch.
