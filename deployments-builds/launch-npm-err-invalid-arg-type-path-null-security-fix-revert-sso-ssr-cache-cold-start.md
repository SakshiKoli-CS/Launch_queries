QUESTION

Customer **Milestone Systems** (Project `68d29b7ceaa2ca1bb35ac4d0`, Azure EU, SF case **00052551**) — **urgent**, first Contentstack app going live next day. Deployment **broke after changing an env variable and redeploying** (no code/project change in a month). Install fails:

```
Installing dependencies...
Current Node.js Version: v22.21.1
npm error code ERR_INVALID_ARG_TYPE
npm error The "path" argument must be of type string or an instance of Buffer or URL. Received null
Installing dependencies failed
Deployment failed: Please try to redeploy the site, or contact support for assistance.
```

After it was fixed, two more issues surfaced: (a) **SSO-gated app stopped redirecting to SSO on Production** (showed authenticated pages without login), though Staging was fine with identical settings; (b) a follow-up question about **app startup delay after inactivity.**

_Keywords: npm ERR_INVALID_ARG_TYPE path must be string Received null, install fails after env var change, NPM_CONFIG_CACHE /tmp/.npm, security fix reverted, deployment broke same code worked last month, SSO bypassed production SSR pages cached, no per-environment cache setting Launch, cache-control routeRules prevent CDN caching, cold start idle, health check warm production go-live._

ANSWER

## 1. `npm ERR_INVALID_ARG_TYPE … "path" … Received null` on install → platform security-fix change (reverted)

- The install failure was **caused by a change that had been deployed for a security-issue fix on the platform.** The team **reverted that change**, and the customer's **redeploy then succeeded.**
- It was **not the customer's env-var change or code** — same code worked the prior month; the redeploy simply re-ran install against the platform state that had the problematic change.
- (A suggested interim env var **`NPM_CONFIG_CACHE=/tmp/.npm`** did **not** resolve it — the real fix was the platform revert.)

## 2. SSO bypassed on Production = SSR pages were cached (app-controlled), NOT a per-env Launch setting

- Symptom: the **SSO-gated app served authenticated pages without redirecting to SSO** on **Production only**; Staging fine; **no env/config difference** in the codebase.
- Cause: **SSR-rendered pages were being cached** (so an authenticated page got served from cache to unauthenticated users). **This is controlled by the application via `Cache-Control` headers — Launch does NOT cache differently per environment.** Every Launch environment is **identical**; there is **no per-instance caching toggle** (and the team did **not** enable anything while fixing the deploy).
- **Fix (customer-side):** add **`routeRules` / `Cache-Control` headers** to **prevent CDN caching of authenticated/SSR pages** (e.g. `no-store`). Customer applied this and it worked.
- Why "only Production"? Caching/traffic patterns differ by env, so the cached-auth-page problem surfaced where it got hit — **not** because Launch configured that env differently.

## 3. App startup delay after inactivity = cold start; keep prod warm via health-check

- A delay after no traffic is an expected **cold start** — idle apps transition to an idle state, common in **non-production** (uneven traffic); **typically doesn't impact production.**
- For production, a **health-check runs every minute once live** to monitor and **keep functions warm.** It's **set up by a Launch engineer during the go-live call** — for a **self go-live, the customer must tell Launch** so this monitoring/warming is configured (currently a **manual** setup).

## Generalized guidance (for future similar queries)

- **`npm ERR_INVALID_ARG_TYPE "path" … Received null` (or other sudden install breakage) right after a redeploy, with unchanged code** → suspect **platform-side build-image/change drift**, not the customer's env-var/code. Escalate; the fix may be a **platform revert** (cf. SST `$USER`, electron glitch — redeploy re-runs install against current platform state).
- **Authenticated/SSO pages leaking to unauthenticated users (esp. on one env)** → **SSR pages are being cached.** **Launch has no per-environment cache setting** — caching is **app-controlled via `Cache-Control`**. Fix: **`no-store`/route rules** on authenticated/SSR routes so they're never cached.
- **"Why does only one environment behave differently?"** → environments are **identical** on Launch; differences come from **traffic/caching patterns or app config**, not hidden per-env Launch settings.
- **Startup delay after idle** = **cold start**; for production set up the **per-minute health-check (keeps functions warm)** — **self-go-live customers must request it** (manual setup by a Launch engineer).
- Related: [end-user SSO via Edge Functions/env vars](../integrations/launch-end-user-sso-auth-provider-edge-functions-env-vars.md), [stale content / Cache-Control](../caching-cdn/launch-stale-published-content-cdn-cache-no-cache-control-headers-next.md), [idle environment Warm-Up Screen](../logs-monitoring/launch-idle-environment-sleep-error-warm-up-screen.md), [sst install $USER](launch-sst-install-build-failure-user-cgo-error-user-root-avoid-sst-in-build.md).

## Status

- **Resolved (all three).** (1) Install failure = **platform security-fix change, reverted** → redeploy succeeded (not the env var; `NPM_CONFIG_CACHE` didn't help). (2) **SSO bypass = cached SSR pages**; fixed by customer adding **`Cache-Control`/routeRules** — **no per-env Launch cache setting exists**. (3) **Startup delay = cold start**; production stays warm via a **per-minute health-check** set at go-live (**self-go-live → must request it**).
