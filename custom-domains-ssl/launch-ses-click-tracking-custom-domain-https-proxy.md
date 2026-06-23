QUESTION

Customer **Five Iron Golf** (SF case **00059264**) wanted **AWS SES click-tracking** to work on branded custom domains hosted on Launch, for use with **Braze** email. Their tracking domains:

- `click.t.fiveirongolf.com`
- `click.e.fiveirongolf.com`

They implemented a **Launch Edge Function** that reverse-proxies requests to the SES endpoint `r.us-east-1.awstrack.me` with a **Host header rewrite**, deployed to **Production and UAT**. Requests were **not reaching the SES backend** — `curl -sI` showed only `server: cloudflare` and no `x-amzn-requestid` / `x-amzn-trace-id` headers (which they'd seen before). They suspected **Cloudflare was intercepting** requests before the Launch edge function ran, and were confused about who controls the Cloudflare layer.

- **Launch Project:** fig-web-solution — `68adac707f66fc37ed9deb59`
- **Environments:** Production, UAT · **Region:** AWS North America

_Keywords: AWS SES click tracking, awstrack.me, custom tracking domain, Braze, edge function reverse proxy, Host header rewrite, server: cloudflare, Cloudflare proxied vs DNS only, CNAME to SES, HTTPS certificate mismatch, TLS SNI, Next.js catch-all route, app/[...proxy]/route.ts, branded tracking domain SSL._

ANSWER

## Clearing up the Cloudflare confusion

- **`server: cloudflare` is EXPECTED** — Launch itself runs on Cloudflare (its edge service is a Cloudflare Worker). Seeing this header does **not** mean a second proxy layer is intercepting traffic.
- **Contentstack does NOT manage the customer's domain DNS.** Launch only manages the Cloudflare zone for **its own infrastructure**, not `fiveirongolf.com`. The customer (or their registrar/previous vendor) controls their DNS.
- DNS verification confirmed both domains CNAME correctly to Launch:
  ```
  dig click.t.fiveirongolf.com CNAME +short → fig-web-solution-production.contentstackapps.com.
  dig click.e.fiveirongolf.com CNAME +short → fig-web-solution-production.contentstackapps.com.
  ```
  So there was **no rogue second Cloudflare zone** — requests were reaching Launch.

## Recommended solution (the correct architecture)

**For SES click tracking, no Edge Function / proxy / Host rewrite is required.** The validated approach (confirmed working directly against AWS SES):

1. Create a **custom tracking domain in SES**.
2. Associate the hostname with an **SES Configuration Set**, enable **click tracking + an Event Destination**.
3. Point the tracking domain **directly via DNS** to the SES endpoint:
   ```
   click.t.yourdomain.com → CNAME → r.us-east-1.awstrack.me
   ```
4. **Only update the CNAME target** — leave existing **TXT / verification / domain-validation** records unchanged.

SES then handles click tracking + redirection itself. **HTTP tracking worked end-to-end** for the customer once they switched to this direct-DNS model.

## The HTTPS / SSL problem (and its fix)

**Problem:** With the direct DNS-only setup (CNAME → `r.us-east-1.awstrack.me`), **HTTPS requests to the branded domain present a certificate mismatch** — AWS serves a TLS cert for the SES endpoint, not for the branded `click.t.fiveirongolf.com`. Reproduced and confirmed on the Launch side.

**Working approach to get HTTPS on the branded domain WITHOUT Edge Functions** — let Launch terminate TLS and proxy via a Next.js app route:

1. Point the tracking domain's CNAME to the **Launch-provided custom domain value** (not directly to SES).
2. Add a **catch-all dynamic route** in the Next.js App Router — e.g. `app/[...proxy]/route.ts` — to handle all requests on the tracking domain.
3. In the route, **validate** that:
   - the request arrived on the configured tracking domain, and
   - the path matches a valid SES-generated tracking path.
4. Forward valid requests to the **SES tracking endpoint, preserving the branded `Host` header** SES requires; reject/handle others locally.
5. Return the AWS response to the user (e.g. honor SES's redirect to the destination URL).
6. Configure the handler for **all relevant methods** (GET, HEAD, POST, …).
7. **Do not cache** these requests — SES tracking URLs/responses are request-specific. Return `Cache-Control: no-store, no-cache` so every request hits SES directly (caching → stale redirects / wrong tracking).

With this, **Launch handles TLS termination and serves the SSL cert for the branded domain**, while SES still does the actual tracking/redirection. Validated working over HTTPS.

**Implementation note (why a Node route, not Edge):** the sample used the **Node.js runtime**, because the **edge runtime can't set the TLS SNI separately from the Host header** — needed here to present the branded host while connecting to SES:
```ts
import https from "node:https";
import type { NextRequest } from "next/server";
// Node runtime required — edge runtime can't set TLS SNI separately from Host header
export const runtime = "nodejs";
```

## Why the original Edge Function approach failed / was set aside

- The Edge Function (rewrite Host → `r.us-east-1.awstrack.me` and `fetch`) was **confirmed active**, but behavior observed within the **Edge layer affected this flow** — outbound fetch to SES wasn't producing the expected SES headers.
- Guidance was to **avoid the Edge-Function reverse-proxy approach** and use either the direct-DNS model (HTTP) or the **Next.js catch-all route** model (HTTPS, branded cert). An Edge-based solution was still being evaluated as of the thread.

## Final resolution

- Customer switched to the recommended DNS-based model; **tracking worked end-to-end**. The custom domains briefly showed **pending / "Moved"/"Deleted"** domain + certificate status — this was resolved by **refreshing the status in Launch** (showed **Active / Active** after refresh; a prior delete attempt may have caused the stale state). **Case resolved.**

## Generalized guidance (for future similar queries)

- **`server: cloudflare` on a Launch domain is normal** — Launch runs on Cloudflare; it's not evidence of a second proxy. Confirm with `dig ... CNAME +short` that the domain points to the `*.contentstackapps.com` target.
- **Contentstack does not control the customer's DNS/Cloudflare** — only Launch's own infra zone. DNS changes (Proxied→DNS Only, CNAME targets) are the customer's to make at their registrar/DNS provider.
- **SES click tracking does NOT need a Launch Edge Function** — point the tracking domain via **CNAME directly to `r.us-east-1.awstrack.me`** (update only the CNAME; keep TXT/verification records).
- **HTTPS on a branded SES tracking domain** → direct-to-SES gives a **cert mismatch**. To serve a valid branded cert, CNAME to the **Launch custom domain** and proxy via a **Next.js catch-all route** (`app/[...proxy]/route.ts`) that preserves the branded Host header and returns `Cache-Control: no-store, no-cache`.
- **Use the Node.js runtime** for such a proxy route — the **edge runtime can't set TLS SNI independently of the Host header**.
- **Never cache SES tracking requests** — they're request-specific; caching causes stale redirects / incorrect tracking.
- **"Custom domain stuck pending / Moved / Deleted"** → try **refreshing the domain status** in Launch; stale state (e.g. after a delete attempt) can clear to Active/Active on refresh.
