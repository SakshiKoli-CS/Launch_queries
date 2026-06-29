QUESTION

Customer **HUGE** (formerly Hero Digital, SF case **00050941**, High; Project `6888fe93d8181072d8b0e03b`) reports `https://www.myhighstreet.com/` is **redirecting ALL URLs to the homepage**. Started recently (was fine hours earlier), go-live next morning. Notes:

- The internal Launch domain `hsip-website-production.azcontentstackapps.com` **works**; the custom domain **always redirects to `/`.**
- Customer says **no code change** would affect redirects, but they **republished**. Initially suspected **Cloudflare.**
- Later, after go-live, a **WAF question**: high volume of malicious requests (`.env`, `wlwmanifest.xml`, `.php`) — is there a WAF, and are these blocked?

_Keywords: all URLs redirect to homepage, every path redirects to /, Next.js middleware redirect, Accept header text/html */* redirect to root, republish, internal domain works custom domain redirects, Launch WAF DDoS standard platform-wide, no per-user WAF rules, .env .php wlwmanifest blocked, edge function block extensions, bring your own WAF._

ANSWER

## Redirect cause — the app's own Next.js middleware (Accept-header logic)

- The "every path → `/`" behavior was caused by the customer's **Next.js `middleware.ts`**, not Launch/Cloudflare. The middleware **redirected any request whose `Accept` header did NOT contain `text/html` or `*/*` to `/`.**
- That logic matches the observed symptom (assets/non-HTML-ish requests bounced to home). **Removing the middleware resolved it** — both sites were redeployed without it.
- Why the internal `*.azcontentstackapps.com` "worked" while the custom domain didn't is consistent with how the requests/headers were hitting the middleware — the **redirect originated in app code**, not the platform.
- **Triage approach (reusable):** when "all URLs redirect," check the app's **`middleware.ts`, `next.config` redirects, `launch.json`, and any `[proxy].edge.js`** before blaming Cloudflare/Launch. Here it was the **middleware's Accept-header rule.**

## WAF question — Launch WAF is standard & platform-wide, not per-user

- **Launch provides WAF + DDoS protection as a standard setup across the entire platform** — **same for all sites**; it does **NOT offer custom/per-user/per-environment WAF rules.**
- The flagged requests (`.env`, `wlwmanifest.xml`, `.php`) were **genuine internet background noise** (bots probing for WordPress/PHP/env files that don't exist on a Node app) — they **404** harmlessly. **Launch does not block these extensions**, because some projects legitimately serve such paths.
- **To block specific requests/extensions yourself:** use a **Launch Edge Function.**
- **For enterprise/custom WAF needs:** put **your own WAF provider in front of Launch** (usually unnecessary, but supported).

## Generalized guidance (for future similar queries)

- **"All URLs / every path redirect to the homepage"** → almost always **app-side redirect logic**, not Launch/Cloudflare. Inspect **`middleware.ts` (e.g. an `Accept`-header check redirecting to `/`), `next.config` redirects, `launch.json`, `[proxy].edge.js`.** A **republish/redeploy** can surface previously-dormant logic. Removing/fixing the offending rule resolves it.
- **"Is there a WAF? Are `.php`/`.env`/`wlwmanifest.xml` blocked?"** → Launch has a **standard platform-wide WAF + DDoS** (no per-user rule customization); those probe requests are **harmless bot noise that 404** and are **not blocked** by default. **Block them via an Edge Function**, or **bring your own WAF** in front of Launch for enterprise needs.
- Related: [redirect loop cached 1 year](../caching-cdn/launch-redirect-loop-308-cached-redirect-smaxage-1year-cache-control.md), [403 programmatic access — Cloudflare WAF/Bot](../networking-connectivity/launch-403-programmatic-access-cloudflare-waf-bot-browser-integrity-check.md), [intermittent deploy failures (same HUGE project)](../deployments-builds/launch-intermittent-deployment-failed-setting-up-site-pipeline-live-unaffected.md).

## Status

- **Resolved.** "All URLs → homepage" was the app's **Next.js middleware** redirecting non-`text/html`/`*/*` Accept-header requests to `/`; **removing the middleware fixed it.** **WAF clarified:** Launch's **WAF/DDoS is standard platform-wide, no per-user rules**; probe traffic (`.php`/`.env`/`wlwmanifest.xml`) is **harmless 404 bot noise, not blocked** — use an **Edge Function** to block, or **own WAF** for enterprise needs.
