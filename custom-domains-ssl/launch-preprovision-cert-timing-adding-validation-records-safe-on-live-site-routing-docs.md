QUESTION

Customer asks two pre-go-live questions:
1. **Pre-provisioning certificates before go-live** — how long does it take, and can they **add the "TXT records to verify domain ownership" NOW without breaking their current live sites**?
2. **URL path routing / Edge URL Rewrites / Edge Functions** — is there documentation/guidance so they feel comfortable with the path forward (or do they need a session)?

_Keywords: pre-provision certificate before go-live how long, add TXT CNAME validation record now safe live site, does adding validation record affect live traffic, certificate issuance time minutes to hours DNS propagation, only update A/CNAME after cert active, edge url rewrites redirects edge functions documentation, regex routing rewrites, request support session walkthrough._

ANSWER

## 1. Pre-provisioning certs — timing + it's SAFE to add validation records to a live site

**Process:**
1. **Create the custom domain in Launch.**
2. **Get the DNS validation record from Support** (typically a **CNAME or TXT**).
3. **Add that validation record to your DNS.**
4. **Wait for the certificate to be issued + marked active.**
5. **Only AFTER the cert is active**, update your DNS (main **A/CNAME**) to point traffic to Launch's CDN.

**How long?** Once the validation record is in place, **certificate issuance is usually fast — minutes to a few hours** — but exact timing depends on **DNS propagation** and the **certificate authority's response**.

**Can you add the TXT/CNAME validation record NOW, on a live site?** **Yes — safely, at any time, with NO impact on the live site**, *as long as you do NOT modify the main A/CNAME records that direct traffic.* The validation record **only verifies domain ownership for cert issuance**; it **does not route/affect live traffic**. So pre-provisioning can be done well ahead of the cutover, decoupled from the actual DNS switch.

## 2. Routing / Edge URL Rewrites / Edge Functions — documentation

- Comprehensive docs exist for **Edge URL Rewrites**, **Edge URL Redirects**, and **Edge Functions** (setup, syntax, best practices).
- **Rewrites** → flexible, **regex-based routing**. **Edge Functions** → more advanced/dynamic logic. The **Go-Live Guide** also summarizes the routing options.
- For **custom scenarios or a walkthrough**, request a **Launch support session**.

## Generalized guidance (for future similar queries)

- **"Can I add the domain-validation TXT/CNAME now without breaking the live site?"** → **Yes, always safe** — validation records only prove domain ownership for cert issuance and **don't affect traffic**. The **only** step that moves traffic is changing the **main A/CNAME**, which you do **after the cert is active**. This is exactly what enables **zero-downtime pre-provisioning**.
- **Cert issuance time** ≈ **minutes to a few hours** after the validation record is live (subject to DNS propagation + CA). Pre-provision **early** so go-live is just the A/CNAME flip.
- **Routing:** **Edge URL Rewrites** (regex routing), **Edge URL Redirects**, **Edge Functions** (advanced logic) — docs cover it; offer a **support session** for custom cases.
- Related: [multi-domain go-live full process (www Cloudflare / apex Fastly)](launch-multi-domain-golive-apex-fastly-www-cloudflare-full-cutover-process-dns-records-bfs.md), [go-live support request in advance + standard steps](launch-golive-support-request-in-advance-standard-steps-schedule-weekday-ist-pods.md), [cutover cert + hostname validation green before cutover](launch-cutover-lets-encrypt-cert-hostname-validation-txt-apex-vs-www-jax.md), [domain-limit increase + per-subdomain DCV/hostname records](launch-domain-limit-increase-golive-provide-dcv-txt-cname-add-subdomain-cne.md), [multi-domain one stack + per-domain routing + Edge Redirects](launch-multi-domain-one-stack-per-domain-content-routing-apex-manual-edge-redirects-dashin.md).

## Status

- **Answered.** **Cert pre-provisioning:** create custom domain → get validation record (CNAME/TXT) from Support → add to DNS → wait for cert **active** (usually **minutes to a few hours**, DNS-propagation/CA dependent) → **then** flip the main A/CNAME. **Adding the validation record now is SAFE on a live site** (doesn't touch traffic — only the A/CNAME switch does). **Routing:** pointed to **Edge URL Rewrites / Redirects / Edge Functions** docs + Go-Live Guide; **support session** available for custom scenarios.
