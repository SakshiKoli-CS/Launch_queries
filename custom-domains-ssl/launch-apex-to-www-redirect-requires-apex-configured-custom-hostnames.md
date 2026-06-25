QUESTION

Customer **Arkansas Blue Cross Blue Shield** (Skai site) moved their site to Launch but **can't get the apex domain `https://skaibluecross.com` to redirect to `https://www.skaibluecross.com`**. They ask what to check on the Contentstack/Launch end to resolve it.

_Keywords: apex to www redirect, apex domain not redirecting, skaibluecross.com to www, apex not configured in Launch, Custom Hostnames add verify apex, redirect apex to subdomain Launch UI, naked domain redirect._

ANSWER

## Cause — apex domain isn't fully configured in Launch

- Review showed the **apex domain (`skaibluecross.com`) is not fully configured in Launch**, so the apex→www redirect can't take effect.

## Fix

1. **Add and verify the apex domain** under **Custom Hostnames** in Launch (per the official Custom Hostnames documentation).
2. Once the apex is configured/verified, **set up the redirect from the apex to the `www` subdomain directly within the Launch UI.**

## Generalized guidance (for future similar queries)

- **"Apex (naked) domain won't redirect to www"** → first check that the **apex domain itself is added and verified in Launch under Custom Hostnames.** The **apex→www redirect can only be configured after the apex is set up** — it's a built-in **Launch UI redirect**, not something the customer needs to build.
- Order matters: **configure + verify apex → then add the redirect in the UI.** A missing/partial apex config is the common reason the redirect "doesn't work."
- Related (apex serving an old/expired cert because it was handled externally rather than in Launch): [apex cert renewal / move apex to Launch](launch-apex-cert-renewal-expired-redirect-move-apex-to-launch.md). Related (apex needs to be created in UI before A record/cert): [custom domain apex no A record](launch-custom-domain-apex-no-a-record-ssl-not-active.md).

## Status

- **Answered.** Cause: **apex domain not fully configured in Launch** → add + verify under **Custom Hostnames**, then configure the **apex→www redirect in the Launch UI**. Relayed to the customer; awaiting their action/confirmation.
