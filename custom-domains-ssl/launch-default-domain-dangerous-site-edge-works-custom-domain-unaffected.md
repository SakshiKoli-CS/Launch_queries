QUESTION

Customer **Service Foods** (SF case **00058719**) reported a Chrome **"Dangerous Site / Attackers might be trying to steal your information"** warning on their Launch **default domains**:
- `https://sf-contentstack-cms.contentstackapps.com/`
- `https://sf-contentstack-cms-uat.contentstackapps.com/`

Observations:
- Started **yesterday**; reproducible for **all internal users and clients**.
- Seen mainly in **Google Chrome**; **works correctly in Microsoft Edge**.
- Customer also mentioned **intermittent application / security-trust errors on custom domains**.

_Keywords: dangerous site warning, Google Safe Browsing, contentstackapps.com flagged, default domain, works in Edge, custom domain trust error, attackers stealing information, Chrome warning, phishing flag._

ANSWER

## Cause

- The **default `*.contentstackapps.com` domains** were flagged by **Google Safe Browsing** → Chrome's "Dangerous Site" warning. Platform-level flag on the **shared default domain**, not a Service Foods compromise. (Same recurring issue — see related entries below.)

## Browser behavior

- The warning shows only in **browsers/services that use the Google flag** (Chrome, and others backed by Google Safe Browsing).
- **Microsoft Edge works correctly** here (it didn't flag the domain), confirming the site itself is healthy — the issue is the **Safe Browsing flag**, not the application.

## Custom domains are generally NOT impacted

- This flagging **typically does not impact custom domains** added by users — **unless the actual site/domain genuinely contains phishing or suspicious behavior**.
- So the intermittent **security-trust errors the customer saw on custom domains** are **separate** from the default-domain Safe Browsing flag and should be investigated on their own (don't conflate the two).

## What to tell the customer

- The "Dangerous Site" warning is limited to the **flagged default domain** on Google-backed browsers; it does **not** mean the customer's site or their custom domains are unsafe.
- Clearing the flag is a **platform/Security-team action**; for production, use a **custom domain** (not subject to these default-domain flags).

## Generalized guidance (for future similar queries)

- **"Default domain shows Chrome 'Dangerous Site' but Edge is fine"** → classic **Google Safe Browsing flag on `*.contentstackapps.com`**; Edge/Firefox not using Google won't show it. Platform clears the flag; site is otherwise healthy.
- **Custom domains are not affected** by the default-domain flag unless they actually host phishing/suspicious content — treat custom-domain trust errors as a **separate** investigation.
- **Prevention / long-term:** keep default domains `noindex`/blocked from Google's scanner and serve production on **custom domains**.
- Related (same root cause): [launch-default-domain-google-safe-browsing-dangerous-site-flag.md](launch-default-domain-google-safe-browsing-dangerous-site-flag.md) and [launch-contentstackapps-dangerous-site-noindex-robots-assets-blocked.md](launch-contentstackapps-dangerous-site-noindex-robots-assets-blocked.md).

## Status

- Customer informed that the warning is limited to Google-flagged default domains and does not indicate an unsafe site; flag clearing handled platform-side. Custom-domain trust errors to be looked at separately if they persist.
