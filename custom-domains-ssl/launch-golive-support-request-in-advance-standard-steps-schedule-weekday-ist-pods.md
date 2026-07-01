QUESTION

Customer **PODS** (UID `bltf391054f55c32754`, with SI **Xcentium**) is going live and wants **Launch team support for the apex-domain switch/cutover** and a **pre-go-live call** with Launch + Xcentium + PODS infra. What are the **standard go-live steps to share**, and **how should go-live-call support be requested/scheduled** so the Launch team can plan?

_Keywords: go-live support request Launch team join cutover call, apex domain switch help, schedule go-live call in advance, weekday overlap with IST timezone, standard go live steps share with customer, reduce TTL, add subdomain Launch UI production environment, Launch provisions and renews certs, TXT records ahead of time, A record CNAME during cutover, Go Live guide reference, no dependency on Launch team late-night calls._

ANSWER

## Standard go-live steps to share with the customer

1. **User reduces the TTL** on their DNS records (so cutover propagates fast).
2. **User adds the subdomain** (e.g. `www.example.com`) in the **Launch UI**, on the **correct production environment**.
3. **Launch team enables the redirect** from apex (`example.com`) → `www.example.com`.
4. **Launch team shares TXT records** for each site going live → the **user adds them at their DNS provider**. This lets Launch **provision the HTTPS certs ahead of time** (Launch **manages the certs and renewals**).
5. **User adds the DNS records and notifies Launch.**
6. **Launch confirms cert provisioning succeeded** and tells the customer they're **OK to go live anytime**.
7. **CSE/CSM sets up the go-live call** and adds the Launch team (schedule on a **weekday at a time that overlaps IST**).
8. **Go-live:** Launch **joins the call** and **shares the A record + CNAME records** the customer updates in their DNS **during the cutover**.

Always point the customer at the **Launch "Go Live" guide** as the reference.

## How to request go-live-call support (the process point of this thread)

- **Schedule the go-live call in advance, on a weekday, overlapping IST** — as outlined in the Go Live guide. Late-night / same-night requests (e.g. 11:30 PM ET / very late IST) make it hard for the team to plan and staff.
- Provision **certs ahead of time** (steps 1–6) so the actual go-live is just the **A/CNAME flip** — often there's then **no hard dependency on Launch being live on the call**; Launch joins as backup and the customer emails if issues arise.

## Generalized guidance (for future similar queries)

- **Customer asks for Launch support on an apex/domain cutover** → share the **8-step go-live flow** above and the **Go Live guide**; the technical record details (www vs apex, DCV, which record to flip when) are in the related full-process entry.
- **Emphasize advance scheduling:** go-live calls should be requested **ahead of time, on a weekday, overlapping IST**, so the team can plan. Same-night/off-hours asks risk no coverage.
- Because **certs are pre-provisioned**, go-live itself is usually just the **DNS flip (A record + www CNAME)** — Launch on the call is a **safety net**, and frequently there's **no blocking dependency** on them.
- Related: [multi-domain go-live full process (www Cloudflare / apex Fastly, records + timing)](launch-multi-domain-golive-apex-fastly-www-cloudflare-full-cutover-process-dns-records-bfs.md), [cutover cert + hostname validation green before cutover](launch-cutover-lets-encrypt-cert-hostname-validation-txt-apex-vs-www-jax.md), [cert pending → DCV needs CNAME not TXT; request go-live support in advance](launch-cert-pending-dcv-needs-cname-not-txt-domain-limit-golive.md), [PODS apex down TLS/SAN](launch-apex-down-tls-san-misconfig-100-domain-limit-fastly-apex-redirect.md).

## Status

- **Completed — went live smoothly (Oct 14).** Shared the **standard go-live steps + Go Live guide**; Launch **joined the cutover call**. With **certs pre-provisioned**, there was **no hard dependency on the Launch team** during cutover (they joined as support; customer to email if needed). Repeated takeaway: **request/schedule go-live calls in advance (weekday, IST overlap)** per the Go Live guide so the team can plan.
