QUESTION

Customer **Motorola Solutions** (SF case **00059173**, P1) could not integrate their Launch environments with infrastructure that uses **Imperva as the WAF layer** with custom DNS routing. Routing traffic through Imperva to the **default `*.contentstackapps.com` Launch domains** produced **Cloudflare Error 1000 ("DNS points to prohibited IP")** across multiple environments — a routing conflict from **nested proxying** (Imperva → Cloudflare → Launch).

They had tried DNS-only (grey-cloud) records, pointing DNS to Imperva endpoints, and TXT/DCV validation, but still hit issues. They asked about moving to **custom domains**: setup, routing expectations behind Imperva, behavior differences vs default domains, origin-access considerations, and whether default domains could be removed.

_Keywords: Imperva WAF, Cloudflare Error 1000, DNS points to prohibited IP, nested proxying, external WAF/CDN in front of Launch, Host header override, custom domain origin, contentstackapps.com behind proxy, O2O Orange-to-Orange, WAF integration, default domain suppression._

ANSWER

## Root cause — nested proxying conflict

Launch is **natively built on Cloudflare** (its CDN/WAF). When **Imperva sits upstream** and routes to the **default `contentstackapps.com` domains** (which resolve to Cloudflare IPs), Cloudflare sees a request arriving from a proxy pointing back at itself → **Error 1000 ("DNS points to prohibited IP")**.

- **Cloudflare's Orange-to-Orange (O2O) workaround does NOT apply** — Imperva is not Cloudflare, and O2O only works when the customer uses their **own Cloudflare zone**. Hard constraint.
- **Conflicting flow:** `User → Imperva WAF → Cloudflare (Launch CDN) → Launch Origin`.
- **Routing through Imperva to default domains is an unsupported path.** Custom domains are the required approach.

## ✅ Actual resolution (confirmed by customer)

**The fix was a Host Header Override rule on the WAF (Imperva) side, pointing at the Launch origin.** Once applied, tested, and deployed, all environments loaded correctly — verified across multiple environments with different configs (auth on/off, different proxy settings).

**Why this works:** Launch resolves environment config (redirects, rewrites, KV store) from the **exact `Host` header**. When an upstream proxy like Imperva forwards a request, the Host header must carry the value Launch expects for that environment, or routing fails. The Host Header Override ensures Imperva presents the correct host to the Launch origin.

> This is the key takeaway for any "Launch behind Imperva / external WAF" case: **set a Host Header Override to the Launch origin host.**

## Recommended architecture (Imperva + Launch custom domains)

**1. Create a custom domain per environment** (Environment → Settings → Domains → + New Domain). Launch auto-populates a **CNAME target** (e.g. `xxxx.contentstackapps.com`) to use as the **Imperva origin backend**. Default domains can't work cleanly behind Imperva because they share Cloudflare routing that conflicts with upstream proxies, risk cache-key collisions across environments, carry automatic `X-Robots-Tag: noindex`, and may be flagged by Google Safe Browsing.

**2. Pre-provision SSL for zero downtime (DCV via TXT):** create the custom domain → get TXT validation records from Support → add to DNS (Launch issues the TLS cert silently) → only switch routing after the cert is active.

**3. Configure Imperva correctly:**
- **Forward the `Host` header to the Launch origin host** (the override above) — Launch uses the exact Host to resolve env config; wrong/missing = routing fails entirely.
- **Disable Imperva caching** (or purge after every deploy) — Launch auto-purges its CDN on each deploy; Imperva won't get those signals → stale content. Best practice: let Launch manage caching.
- **Imperva owns WAF/DDoS** — Launch's built-in WAF is bypassed once an upstream proxy intercepts traffic, so all firewall/DDoS rules must live on Imperva.

**4. Suppress (not delete) default domains:** Default domains **cannot be deleted** at the platform level (known constraint). Suppress them with a **Launch Edge Function** that checks the Host header and returns a 301/403 for any `*.contentstackapps.com` request. (Launch already sets `X-Robots-Tag: noindex, nofollow` on default-domain responses, so SEO impact is contained.)

## Supported vs not supported

- ❌ **Imperva → Launch default domains** — Error 1000, unsupported.
- ✅ **Imperva → Launch custom domains** (correct Host header + Imperva caching off) — supported.
- ❌ **Deleting/disabling default domains** — not available; use an Edge Function redirect instead.
- ✅ **DNS-only CNAME → Launch (no upstream proxy)** — the recommended default path.
- ❌ **Cloudflare O2O** — not applicable unless the customer owns a Cloudflare zone.

## Generalized guidance (for future similar queries)

- **"Cloudflare Error 1000 routing an external WAF/CDN (Imperva, Akamai, etc.) to Launch"** → caused by **nested proxying onto default `contentstackapps.com` (Cloudflare IPs)**. Fix: route to a **Launch custom domain** as origin and set a **Host Header Override** to the Launch origin host on the WAF.
- **Host header is the linchpin** — Launch resolves per-environment config from the exact Host; an upstream proxy must forward/override it correctly.
- **Disable upstream caching** (Launch auto-purges on deploy; upstream caches go stale).
- **Default domains can't be deleted** — suppress via an Edge Function 301/403 on the Host header.
- **O2O only with the customer's own Cloudflare zone** — not for third-party WAFs.

## Status

- **Resolved.** Customer fixed it on their **WAF side via a Host Header Override** for the Launch origin; environments work across configs. A review call was offered to confirm the final configuration. (Candidate for a docs update on "Launch behind an external WAF / Imperva".)
