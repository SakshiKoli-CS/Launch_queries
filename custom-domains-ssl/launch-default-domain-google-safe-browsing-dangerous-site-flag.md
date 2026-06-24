QUESTION

A **Google Safe Browsing "Dangerous site" warning** was appearing on Launch default domains (`*.contentstackapps.com`), affecting **customer-facing** pages and the **support portal**. Multiple reports:
- **XCentium** (case **00058586**) — "Dangerous site" flag warning for **West Monroe** on Launch
- **Decathlon** (case **00058733**) — impacted **support portal**
- **BSH** — hit the security warning page

The browser blocked the page with the interstitial warning before users could reach the site.

_Keywords: Google Safe Browsing, dangerous site warning, deceptive site, contentstackapps.com flagged, default domain blocked, security interstitial, support portal blocked, Chrome warning, Firefox no safe browsing, Cloudflare incident._

ANSWER

## What's happening

- The **default `*.contentstackapps.com` domains** were flagged by **Google Safe Browsing**, so Chrome (and other Safe-Browsing-based browsers) showed a **"Dangerous site"** interstitial. This is a **platform-level flag on the shared default domain**, not a per-customer compromise.
- Default domains are known to be **periodically flagged by Google Safe Browsing** (custom domains are not affected — see related guidance below).

## Temporary workarounds (during the incident)

- **Click "Details" on the warning page** and proceed via the link there.
- **Use Firefox**, which does not use Google Safe Browsing, so it doesn't show the interstitial.
- ⚠️ These are **not acceptable as a customer-facing fix** — they're stopgaps while the flag is cleared at the platform level.

## Why this is a platform action (not customer-fixable)

- Clearing the Safe Browsing flag and the related domain/security changes require **DevOps access** at the platform level; customers/support can't resolve it on their side.
- During this incident there were also **concurrent Cloudflare issues** (Cloudflare status incident) that had to resolve first — Launch runs on Cloudflare, so the fix had to wait on that.

## Long-term avoidance — use custom domains for production

- **Production sites should use a custom domain**, not the default `*.contentstackapps.com` URL. Default domains:
  - are periodically **flagged by Google Safe Browsing**,
  - carry automatic **`X-Robots-Tag: noindex, nofollow`** headers (bad for production SEO).
- Custom domains avoid both. (See [launch-imperva-waf-cloudflare-error-1000-host-header.md](../networking-connectivity/launch-imperva-waf-cloudflare-error-1000-host-header.md) for the same default-domain caveats in a WAF context.)

## Generalized guidance (for future similar queries)

- **"Our Launch site shows a Google 'Dangerous site' warning"** → if it's on a **`*.contentstackapps.com` default domain**, it's a **platform-level Safe Browsing flag** (shared domain), cleared by the platform/DevOps — not a customer compromise.
- **Stopgaps:** "Details → proceed", or Firefox (no Safe Browsing) — but **never present these as the customer-facing solution.**
- **Real fix / prevention:** move production traffic to a **custom domain**; default domains are subject to Safe Browsing flags and `noindex`.
- Check for a **concurrent Cloudflare incident** — Launch is on Cloudflare, and platform-side remediation may be gated on Cloudflare recovery.

## Status

- Raised with the **security team**; remediation required **DevOps** and was gated on a **concurrent Cloudflare incident** resolving. Subsequently **resolved** (verified by the reporter); confirm no pending platform actions remain.
