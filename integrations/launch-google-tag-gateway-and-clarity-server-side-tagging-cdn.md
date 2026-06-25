QUESTION

Customer **Xcentium** (on behalf of **West Monroe**) asks two related third-party-tracking questions for a Launch-hosted site:

1. **Google Tag Gateway** — Google's new **server-side tagging** feature. Normally they install tracking via **GTM (client-side)**; this update runs **server-side and requires installing on the CDN**. They don't think they have CDN access on Launch. **Can they install Google Tag Gateway on the CDN?** ([Google docs](https://developers.google.com/tag-platform/tag-manager/server-side))
2. **Microsoft Clarity** — similar question, how to set it up on a Launch site. ([Clarity setup docs](https://learn.microsoft.com/en-us/clarity/setup-and-installation/clarity-setup))

_Keywords: Google Tag Gateway, server-side tagging, install on CDN, GTM client side to server side, Launch CDN access, third-party server runtime, first-party proxy edge function, CORS, Microsoft Clarity setup, tracking script head, CSP allowlist clarity.ms._

ANSWER

## Google Tag Gateway — Launch won't host the runtime, but fully supports integrating with it

- **Google Tag Gateway is a server-side tagging runtime that must be deployed in infrastructure your org owns/manages** (e.g. your own **GCP account** or Google's managed option).
- **Launch does not deploy or manage third-party server runtimes inside its cloud** — so you **can't install the Gateway "on the Launch CDN."**
- **But there's no platform limitation preventing server-side tagging** — Launch supports **two integration patterns:**

### Option 1 — Third-party setup (recommended default, simplest)

Browser → `https://tags.yourcompany.com` → Google Tag Gateway → Google services

```js
fetch("https://tags.yourcompany.com/collect", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ event: "page_view", page: window.location.pathname })
});
```
- The Launch-hosted site makes a **direct outbound request** to your Gateway. **Launch does not block or modify outbound requests** (validated outbound calls from a Launch site to external endpoints — they work).
- Your **Gateway must allow your site domain in its CORS config.**
- **No CDN changes, no Edge logic** required.

### Option 2 — First-party setup (proxy pattern, optional)

Route tracking through your own domain (e.g. `/collect`) instead of calling the Gateway domain directly:

Browser → `https://www.yoursite.com/collect` → **Launch Edge Function (proxy)** → `https://tags.yourcompany.com` → Google Tag Gateway

```js
// Edge Function proxy
export default async function handler(request) {
  const body = await request.text();
  return fetch("https://tags.yourcompany.com/collect", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body
  });
}
```
```js
// Client call to the first-party endpoint
fetch("/collect", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ event: "page_view", page: window.location.pathname })
});
```

**Summary:** Gateway lives in **your** infra; **Launch hosts the site, not the Gateway runtime**; integrate **directly (third-party)** or via an **optional first-party Edge proxy.**

## Microsoft Clarity — standard manual script install works

1. Create a project in the **Clarity dashboard**.
2. Copy the **unique tracking script** for your project.
3. Add the script inside the **`<head>`** of your site's HTML.
4. **Redeploy** so the change goes live → Clarity starts collecting data.
- **If you have a strict Content Security Policy (CSP)**, **allowlist `https://www.clarity.ms`** in CSP, otherwise the browser blocks the script and Clarity collects nothing.
- Other install options in the Clarity docs work too.

## Generalized guidance (for future similar queries)

- **"Can we install <server-side tagging / a third-party runtime> on the Launch CDN?"** → **No** — Launch **doesn't host third-party server runtimes**; deploy those (e.g. **Google Tag Gateway**) in **your own infra**. But **server-side tagging is fully supported** via either a **direct outbound call** to your gateway (no CDN/edge changes; mind the **gateway's CORS**) or a **first-party Edge Function proxy** on your domain.
- **Launch does not block/modify outbound requests** — direct client→external-gateway calls work.
- **Client-side analytics (Clarity, GA, etc.)** = standard **`<head>` script + redeploy**; if a **CSP** is set, **allowlist the vendor domain** (e.g. `www.clarity.ms`) or the script is blocked.
- Related (edge proxy / outbound patterns): [branded click-tracking reverse proxy](../custom-domains-ssl/launch-branded-click-tracking-reverse-proxy-rewrites-requirements.md), [SES click tracking proxy](../custom-domains-ssl/launch-ses-click-tracking-custom-domain-https-proxy.md).

## Status

- **Answered.** **Google Tag Gateway:** can't be installed on Launch's CDN (Launch doesn't host third-party runtimes) — deploy in your own infra and integrate via **direct outbound** (recommended) or a **first-party Edge proxy**; no platform limitation. **Clarity:** manual **`<head>` script + redeploy** (allowlist `www.clarity.ms` if CSP is enforced). Awaiting customer confirmation.
