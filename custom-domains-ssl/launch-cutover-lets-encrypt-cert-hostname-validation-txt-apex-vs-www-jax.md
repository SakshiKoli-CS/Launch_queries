QUESTION

Customer **The Jackson Laboratory / Jax.org** (SF case **00050006**) hit a **Let's Encrypt certificate error** during a cutover to Launch (Edge) and **rolled back**. They want to **set up a certificate ahead of the next go-live** (a **private key must be entered → working session required**), and ask for **support presence on the cutover call.** Additional questions during setup: **why redirect the apex (`jax.org`) to a subdomain (`www.jax.org`)?** — they need to **keep the apex for other non-web services**.

_Keywords: Let's Encrypt cert error cutover, private key working session, custom certificate setup, hostname validation _cf-custom-hostname TXT record, front.jax.org www.jax.org, provision TLS before go live green check, apex to www redirect benefits, apex single point of failure cookies SEO, custom domain limit increase temporary no cost, missed TXT record cutover broken._

ANSWER

## Get certs + hostname validation GREEN before the cutover (don't cut over half-validated)

- **Cutting over before certs/hostname validation are fully green causes the broken-cert state** they hit. The customer **attempted cutover without adding the required TXT records** → repeated cert issues. **Prefer a "green check" from Launch first.**
- Setup steps per domain:
  1. **Add the custom domain** (e.g. `www.jax.org`) in the relevant **Launch environment**.
  2. **Install the custom certificate** (private key entry → do this in a **working session**; needs the customer's IT).
  3. **Validate the hostname** by adding a **`_cf-custom-hostname.<domain>` TXT record** at the DNS provider, e.g.:
     ```
     _cf-custom-hostname.www.jax.org   TXT   f1d99b9b-…
     _cf-custom-hostname.front.jax.org TXT   08d46253-…
     ```
     Adding these **has no impact on the existing setup** — it's only for Launch-side hostname validation. **Each domain/subdomain needs its own TXT.**
  4. **Confirm** → Launch validates certs + hostname; only then is the domain "set to receive traffic" at go-live.

## Domain-limit increase — temporary, no cost

- A **custom-domain limit increase** was done to unblock the go-live. **A temporary increase like this has no cost**; formal contract review happens after launch (checks under-provisioning), but it's not billed for the temporary bump.

## Apex (`jax.org`) → subdomain (`www.jax.org`) redirect — why, and it's optional

- The customer can **keep/handle the apex themselves** — sensible when the **apex has many non-web dependencies/services**. Apex handling at their end is fine.
- **Benefits of redirecting apex → `www`:**
  - The **apex/root can become a single point of failure.**
  - **Cookies set at the apex are shared across all subdomains** (`front.jax.org`, `www.jax.org`, …) → **security concern.**
  - Can have **SEO implications.**
- So it's a **recommendation, not a requirement** — route traffic to `www.jax.org` directly and let the customer own the apex if they have other services on it.

## Generalized guidance (for future similar queries)

- **Cert/Let's-Encrypt error at cutover** → **provision certs + validate hostnames BEFORE the DNS cutover**; get a **green check from Launch** to avoid a broken-cert site. A **custom cert with a private key** needs a **working session** with the customer's IT.
- **Hostname validation = a `_cf-custom-hostname.<domain>` TXT record per domain/subdomain** (no impact on existing setup). A **missed TXT record on the domain you're cutting to** = the cutover fails/breaks — verify **every** target subdomain's TXT is added.
- **Temporary domain-limit increases are no-cost** (formal contract review post-launch).
- **Apex→www redirect is recommended** (avoid apex single-point-of-failure, apex cookie sharing across subdomains, SEO) **but optional** — customers with non-web services on the apex can **own the apex themselves** and route web traffic to `www`.
- Related: [apex→www redirect requires apex configured](launch-apex-to-www-redirect-requires-apex-configured-custom-hostnames.md), [add SAN domains / DCV CNAME 7-day](launch-add-san-domains-managed-cert-100-limit-format-renewal-cname-dcv-7day.md), [cert pending → DCV CNAME not TXT](launch-cert-pending-dcv-needs-cname-not-txt-domain-limit-golive.md), [www→apex canonical flip](launch-www-to-apex-canonical-flip-entitlement-increase-outage-redirect-recovery.md).

## Status

- **In progress → guided.** Cert setup done via **working session** (private key); domains **`www.jax.org` + `front.jax.org`** got the custom cert + TLS, pending **`_cf-custom-hostname` TXT hostname validation** (customer repeatedly **missed the `front.jax.org` TXT** → cutover broke). Guidance: **get certs/validation green before cutover.** **Domain-limit increase = no cost (temporary).** **Apex→www redirect recommended (SPOF/cookie/SEO) but optional** — customer keeps the apex for other services; traffic routes directly to `www.jax.org`.
