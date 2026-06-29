QUESTION

Customer **PODS** (Org `bltf391054f55c32754`, Azure NA) reports their **application is down** — specifically **`pods.com` errors while `www.pods.com` works** (the error referenced **Fastly**). Support could access both. This happened **during ongoing work to add their SANs** (a large multi-domain certificate request).

Customer follow-ups: **which TLS/SAN changes were applied, any pending, and a recommended process for production cert updates?** Also: **"Does Contentstack use Fastly? I thought it's Cloudflare CDN — why a Fastly error?"**

_Keywords: apex domain down, pods.com errors www works, Fastly error apex redirect, TLS misconfiguration, SAN certificate work, accidentally removed apex domain, max 100 domains allowed per service, split domains two services, A records to provision certificate, Cloudflare CDN but Fastly for apex._

ANSWER

## Cause — apex domain accidentally removed during SAN work (TLS misconfiguration)

- The outage was a **TLS misconfiguration introduced during the SAN-addition work.** While managing the domain list (which showed **"max 100 domains allowed"**), the engineer **accidentally removed `pods.com` itself** instead of the newly added domain → the **apex went down** while **`www` stayed up.** It was **rectified immediately.**
- This is why only the **apex (`pods.com`)** errored and **`www.pods.com`** was fine — the apex's config was the one disturbed.

## Why a "Fastly" error if Launch uses Cloudflare?

- **Launch uses Cloudflare for CDN**, **but uses Fastly *only* for the apex-domain redirect.** So an **apex** issue can surface a **Fastly**-branded error even though the main CDN is Cloudflare.

## The 100-domains-per-service limit + the fix applied

- A Launch **service allows a max of 100 domains.** The customer's request was **160 domains**, exceeding one service.
- **Resolution:** **split the 160 domains across two services** — **100 on the `pods.com` service** and the **remaining ~60 on `iampods.com`.**

## Answers to the customer's cert questions

1. **Which TLS/SAN changes were applied?** Split the original **160 domains into two groups** — **100 → `pods.com` service**, remainder → **`iampods.com`.**
2. **Any pending SAN updates?** **No.** You only need to **add the required A records** so the certificates can **provision.**
3. **Recommended process for production cert updates going forward?** **No specific process required** at this time.

## Generalized guidance (for future similar queries)

- **Apex (`domain.com`) down but `www` works**, possibly with a **Fastly** error → look at the **apex config / its redirect**: **Launch uses Fastly only for the apex-domain redirect** (Cloudflare for the main CDN), so apex-specific problems can show Fastly errors. A recent **TLS/SAN/domain-list change** is a prime suspect.
- **Max 100 domains per service** — large multi-domain (SAN) certs beyond 100 must be **split across multiple services** (e.g. a second hostname/service for the overflow).
- After cert/SAN changes, **add the required A records** so certs **provision**; verify the apex wasn't accidentally dropped when trimming the domain list to the 100 cap.
- Related: [apex cert renewal / move apex to Launch (Fastly redirect)](launch-apex-cert-renewal-expired-redirect-move-apex-to-launch.md), [add domains / domain-limit increase](launch-add-new-domains-decommission-repurpose-environment.md), [cert pending → DCV + domain limit](launch-cert-pending-dcv-needs-cname-not-txt-domain-limit-golive.md).

## Status

- **Resolved.** Apex (`pods.com`) outage = **apex accidentally removed during SAN work** (hit the **100-domain/service** cap); **rectified immediately**; `www` was never affected. **Fastly** appeared because Launch uses **Fastly only for the apex redirect** (Cloudflare otherwise). SANs were **split 100/`pods.com` + remainder/`iampods.com`**; **no pending updates**, just **add A records to provision**; **no special ongoing process** needed.
