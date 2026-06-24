QUESTION

Customer **MicroStrategy / Strategy** (SF case **00057382**, P1 outage) — users **couldn't log in** on a Launch-hosted Next.js (**App Router + RSC**) app (`community.strategy.com`). The login route hung (endless loading), showed a **400** in console, then after ~2 minutes a **Cloudflare 520/524** timeout. Affected all environments of the project, all users, US + EU.

Key clues:
- **Works locally** with `npm run build` + production env/code.
- The **`www.strategy.com`** site (same Azure B2C login, but **Pages Router, no RSC**) works fine; **only the App Router/RSC app** fails.
- Failing route: `https://community.strategy.com/api/auth/signin?redirect=%2F`. Hitting it with a different redirect param worked but **very slowly**.
- "No code changes in 1–3 months; not deployed since Mar 24."

_Keywords: 524 timeout, 520, login hangs, /api/auth/signin, NextAuth redirect, 301 Transfer-Encoding chunked, no chunk terminator, Content-Length 0, App Router RSC, works locally fails on Launch, 98 second timeout, Azure B2C redirect._

ANSWER

## Root cause — 301 redirect with `Transfer-Encoding: chunked` but no terminator

- On login, the app's Lambda returns a **301 redirect** (to the Azure B2C login page) shaped like:
  ```
  HTTP/1.1 301 Moved Permanently
  Transfer-Encoding: chunked
  Connection: keep-alive
  Location: https://microstrategyb2c.b2clogin.com/...
  ```
- The response uses **`Transfer-Encoding: chunked`** but has **no body and no chunk terminator (`0\r\n\r\n`)**. In HTTP/1.1, a chunked response is only **complete when the terminator arrives**. Since it never does, **Launch's delivery service waits the full ~98 seconds** before giving up → Cloudflare surfaces a **524** (and the earlier 400/520 the customer saw).

## Fix (in the app's Next.js code) — set `Content-Length: 0` on redirects

- For any redirect response, **explicitly set `Content-Length: 0`**. This tells Node **not** to use chunked encoding and signals the proxy the response is complete immediately:
  ```js
  return new Response(null, {
    status: 301,
    headers: { 'Location': redirectUrl, 'Content-Length': '0' },
  });
  ```
- If the redirect comes from **NextAuth** internally, wrap it in **middleware or a custom route handler** that adds `Content-Length: 0` to any **3xx** response before it reaches Lambda's HTTP layer.
- **Confirmed:** the workaround fixed it in Dev, then Production.

## Why it "worked locally" and "suddenly broke" with no code change

- **Locally:** the browser fetches **directly from the Next.js server with no intermediate hops** (AWS gateway, CDN), so nothing enforces the missing chunk terminator — it appears to work.
- **Why now:** no Launch-side change. Likely an **upstream component (AWS/Cloudflare) became stricter** about the missing terminator (no release notes found, can't be ruled out). Next.js has **known RFC-noncompliance around redirects** that produces exactly this shape — and it surfaced only on the **App Router/RSC** app (the Pages Router app on the same B2C login was unaffected).

## Generalized guidance (for future similar queries)

- **"Login/redirect route hangs ~98s then 524/520; works locally"** → suspect a **3xx response with `Transfer-Encoding: chunked` and no chunk terminator**; the proxy waits for a terminator that never comes. **Fix: `Content-Length: 0` on all redirect responses** (or middleware adding it to 3xx).
- **Works locally, fails on Launch** → local has **no intermediate proxy/CDN** enforcing HTTP framing; the platform's edge does. Not necessarily a platform change.
- **App Router/RSC vs Pages Router** difference on the *same* auth flow → points to framework redirect handling, not the auth provider.
- This is **app-side** (Next.js/NextAuth response shaping), not a Launch defect — though "nothing changed" is fair; upstream strictness can change.

## Status

- **Resolved** — `Content-Length: 0` on redirects fixed it (Dev → Prod, verified working). Root cause: non-RFC-compliant chunked redirect from the app; Launch continued investigating why it surfaced now. (See separate deployment-failure issue from the same customer: [launch-deployment-econnreset-sharp-libvips-upgrade.md](../deployments-builds/launch-deployment-econnreset-sharp-libvips-upgrade.md).)
