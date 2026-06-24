QUESTION

Chrome blocked Launch default-domain URLs with a **"Dangerous Site"** warning (*"attackers on this site might trick you into installing software or revealing passwords, phone numbers, or credit card details"*). Reported across multiple newly created Launch sites on **`*.contentstackapps.com`**, e.g.:
- `contentstack-com-2025-self-service.contentstackapps.com/explorer`
- `portwest.contentstackapps.com/en` (brand-new RPR4.2 demo)

Additional symptom: even when the site loaded (via the warning bypass), **assets were not rendering** — Google was also **blocking the assets**.

_Keywords: dangerous site warning, Google Safe Browsing, contentstackapps.com flagged, default domain blocked, deceptive site, assets not rendering, assets blocked, noindex meta tag, robots.txt block scanner, Chrome Safari Google Safe Browsing, Firefox phishing sensor, new Launch site flagged._

ANSWER

## Cause

- The **default `*.contentstackapps.com` domains** were flagged by **Google Safe Browsing**, triggering Chrome's **"Dangerous Site"** interstitial. This is a **platform-level flag on the shared default domain**, recurring on newly created sites — not a per-site compromise.
- When flagged, Google can also **block the site's assets**, so even after bypassing the warning, **assets may fail to render**.

## Browser behavior

- **Chrome and Safari use Google Safe Browsing** → they show the warning.
- **Firefox uses its own phishing sensors** (not Google) → typically does **not** show the warning, so the site loads there.

## Temporary workarounds (not customer-facing fixes)

- Click **"Details"** on the warning page and use the link below it to proceed.
- Test in **Firefox** to confirm the site itself is fine.
- ⚠️ Stopgaps only — the flag must be **cleared at the platform level by the Security team**.

## Prevention — block Google's scanner from default domains

- The recurrence is tied to Google's scanner indexing the **default domains**. Mitigation/prevention: ensure default-domain responses carry **`noindex` meta tags** and/or a **`robots.txt` that blocks Google's scanner** so the default URLs aren't crawled/flagged.
  - (Launch already sets `X-Robots-Tag: noindex, nofollow` on default-domain responses; confirm this is in place, and add `robots.txt` blocking where applicable, as soon as the flag is cleared.)
- **Best long-term fix:** serve production on a **custom domain**, not the default `*.contentstackapps.com` URL — custom domains are not subject to these Safe Browsing flags.

## Generalized guidance (for future similar queries)

- **"New Launch site shows Chrome 'Dangerous Site' warning"** → it's a **Google Safe Browsing flag on the shared `*.contentstackapps.com` default domain**; cleared by the **Security team** at the platform level, not by the customer.
- **Assets not rendering after bypassing the warning** → Google may also be **blocking assets** on a flagged domain; resolves when the flag clears.
- **Chrome/Safari warn (Google Safe Browsing); Firefox doesn't** — useful to confirm the site is otherwise healthy, but not a fix.
- **Prevent recurrence:** keep **`noindex`/`X-Robots-Tag`** on default domains and **block Google's scanner via robots.txt**; move production to a **custom domain**.
- Related incident (same root cause, multi-customer): [launch-default-domain-google-safe-browsing-dangerous-site-flag.md](launch-default-domain-google-safe-browsing-dangerous-site-flag.md).

## Status

- Raised with the **Security team**, who worked to **restore** the flagged domains/assets. Some apps recovered during the incident; clearing is a **platform-side** action.
