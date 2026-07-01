QUESTION

Customer **The Jackson Laboratory / Jax.org** (SF case **00047039**, stack `blt69aefa072a61bf03`) is transitioning `www.jax.org` (currently pointing to their on-prem Netscaler at **`64.147.57.78`**) to route through **Contentstack Edge (Launch)**. Two questions:
1. **Can we target an Edge-specific IP?** (to point their DNS at)
2. Once on Edge, **Edge will need to call back to their on-prem resources** at `64.147.57.78` (referenced as e.g. `origin.jax.org`). What's the proper configuration on Launch's end to support this?

_Keywords: Edge-specific IP target DNS, Launch does not have specific IPs, proxy traffic by custom hostname, go live doc CNAME, edge function call back to origin on-prem netscaler, origin.jax.org backend, Launch app requests backend data not edge callback, verify request from Launch app custom header, simplify architecture app-to-origin._

ANSWER

## Q1 — There is no "Edge-specific IP" to target

- **Launch does NOT have a specific/fixed set of IPs** to point DNS at (inbound or outbound).
- You **route to Launch by adding the custom hostname** and pointing DNS (CNAME) at the Launch target per the **Go Live** flow — traffic is **proxied based on the custom hostname**, not a static IP.
- Ref: **Launch "Go Live" documentation** (add custom domain → validate → DNS).

## Q2 — Don't have Edge "call back" to the origin; have the Launch app call the origin directly

- The customer's proposed architecture (**Edge → `origin.jax.org` on-prem callback**) is **an unnecessary detour** if the front end is hosted on Launch. Traffic that reaches Edge is going to the **front-end app hosted on Launch** anyway.
- **Simpler, recommended architecture:** the **Launch-hosted app requests backend data from `origin.jax.org` as needed** (server-side) — i.e. **app → origin**, not **edge → origin callback**.
- **Securing those origin calls:** since Launch has **no fixed egress IP to allowlist**, have the Launch app **attach a specific header (shared secret) to its requests to the origin**, and have the origin **verify that header** to confirm the request came from the Launch-hosted app. (Header-based verification is the standard substitute for IP allowlisting on Launch.)

## Generalized guidance (for future similar queries)

- **"What Edge/Launch IP do we point DNS at?"** → **Launch has no fixed IPs.** You go live by **adding the custom hostname and pointing DNS (CNAME) at the Launch target** (Go Live doc); traffic is **proxied by hostname**.
- **"How does Edge call back to our on-prem origin?"** → don't architect an **edge→origin callback**. The **Launch-hosted app fetches backend data from the origin directly** (app→origin). Keep the topology flat: DNS → Launch app → origin backend.
- **Securing app→origin when there's no static egress IP** → the app **sends a shared-secret header**; the **origin validates it** (same pattern used to lock down a backend to Launch-only traffic). See related for the no-static-IP → header/auth guidance.
- Related: [no fixed outbound/egress IP to allowlist](launch-no-fixed-outbound-egress-ip-allowlist.md), [no static IP / no VPN → secure DB/API via auth + Cloud Functions](launch-no-static-ip-no-vpn-secure-db-api-via-auth-cloud-functions-azure.md), [CloudFront-over-Launch → block via shared header (origin relationship)](../edge-functions-rewrites/launch-edge-redirect-loop-cloudfront-origin-block-via-header-authorization.md), [cutover cert + hostname validation (same customer, Jax)](../custom-domains-ssl/launch-cutover-lets-encrypt-cert-hostname-validation-txt-apex-vs-www-jax.md).

## Status

- **Answered.** (1) **No Edge-specific IP** — go live by **adding the custom hostname + DNS (CNAME)**; traffic is **proxied by hostname** (Go Live doc). (2) Advised **against an edge→origin callback**: the **Launch-hosted app should request backend data from `origin.jax.org` directly**, and **verify those requests via a custom/shared-secret header** (since there's no fixed egress IP to allowlist). Customer acknowledged.
