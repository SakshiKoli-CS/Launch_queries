QUESTION

For the Contentstack developers website, the team needed `developer.contentstack.com` to **always redirect to `developers.contentstack.com`** (e.g. a 302). They asked whether the Launch team configures this. Follow-up questions:

1. Can the redirect just be done in **Next.js** instead of an Edge Function?
2. If they later add **personalization middleware**, is a **Launch Edge Function faster than native Next.js middleware**?
3. (Operational) The custom domain showed **Pending** — how to make it Active?

_Keywords: subdomain to subdomain redirect, developer vs developers, 302 redirect, apex to subdomain only, custom domain redirect UI limitation, Edge Function redirect, Next.js redirect vs Edge Function, Next.js middleware on edge not supported, personalization middleware, custom domain pending TXT CNAME DCV revalidate._

ANSWER

## The redirect — use an Edge Function (UI redirect won't cover this)

- Add `developer.contentstack.com` as a **custom domain** (Custom Domains docs).
- **Important limitation:** Launch's built-in custom-domain redirect option **only supports apex → subdomain** redirects. Since `developer.contentstack.com` and `developers.contentstack.com` are **both subdomains**, this redirect **cannot be configured through the custom domain UI.**
- After adding + validating `developer.contentstack.com`, implement the redirect with a **Launch Edge Function** returning a **302** (or desired status) to `developers.contentstack.com`. (Edge Functions docs.)

## Edge Function vs doing it in Next.js

- You *can* implement the redirect in **Next.js**, but a **Launch Edge Function is recommended**: it runs **at the edge before the request reaches the application**, giving a **faster response** and **reducing load on the app**.

## Edge Function vs Next.js native middleware (for later personalization)

- For personalization via middleware later, a **Launch Edge Function is generally faster** than native Next.js middleware — it runs at the edge, closer to the user, earlier in the request flow.
- **Key limitation:** **Launch does not support deploying Next.js native middleware on the Edge** — Next.js middleware runs **server-side only** on Launch. (Ref: Next.js on Launch — Limitations.)
- Therefore, for edge-level redirect/personalization use cases, **use Launch Edge Functions**, not Next.js middleware.

## Custom domain activation (Pending → Active)

- A newly added custom domain shows **Pending**. To activate:
  1. Add the required **TXT (hostname validation)** and **CNAME (certificate/DCV validation)** records in your DNS provider, using the values shown in Launch.
  2. Allow time for **propagation** (DNS changes can take up to ~48h depending on provider/TTL); use the **Refresh** button.
  3. Once **both** records are active, add the **final CNAME** record to route traffic to the domain.
  4. If still pending, use the **Revalidate** option from the Actions (three-dots) menu.
- Note: for a Contentstack-owned domain, **DNS is managed internally** (the requester may not have DNS access) — the records were added by the internal DNS owner, after which Hostname + Certificate status went **Active** and the redirect worked.

## Generalized guidance (for future similar queries)

- **"Redirect one subdomain to another on Launch"** → the **custom-domain UI redirect is apex→subdomain only**; for **subdomain→subdomain** (or any non-apex redirect), use an **Edge Function** returning 302/301.
- **"Should I redirect in Next.js or an Edge Function?"** → prefer the **Edge Function** — handled at the edge before the app, faster and lighter on the app.
- **"Is a Launch Edge Function faster than Next.js middleware?"** → yes for edge use cases, and importantly **Launch doesn't run Next.js native middleware at the edge** (server-side only) — so use Edge Functions for edge-level redirects/personalization.
- **Custom domain stuck Pending** → add **TXT + CNAME (DCV)** records, wait for propagation, **Refresh**, then **Revalidate** from the Actions menu; add the final routing CNAME once validated.

## Status

- **Resolved.** Domain validated and active; `https://developer.contentstack.com/` redirects to `https://developers.contentstack.com/` as expected.
