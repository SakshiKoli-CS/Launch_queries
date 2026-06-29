QUESTION

Customer **Wedgewood** (SF case **00052248**) is preparing to go to production on Launch and, for **SEO / canonical URL** reasons, wants **`https://wedgewood.com` redirected to `https://www.wedgewood.com`** — and **all URLs under the domain** too. Is this possible?

_Keywords: canonical URL, apex to www redirect, naked domain to www, SEO canonical, redirect all URLs under domain, wedgewood, already redirecting, 301 308 redirect response headers._

ANSWER

## Yes — apex→www redirect for canonical URLs is supported (and here it was already configured)

- **Redirecting the apex (`wedgewood.com`) to `www.wedgewood.com`** — including **all paths under it** — is supported and is the standard way to enforce a **canonical URL** on Launch.
- In this case the setup was **already redirecting `https://wedgewood.com` → `https://www.wedgewood.com`**, so the **canonical behavior was already achieved.** Confirmed via the **network tab** (redirect + response headers).

## Generalized guidance (for future similar queries)

- **"Can we redirect apex → www for canonical/SEO (and all sub-paths)?"** → **Yes.** Configure the **apex→www redirect** in Launch; it applies to the whole domain (all paths), giving a single canonical host.
- **Verify before assuming it's missing** — check the **network tab / response headers** (a `301`/`308` from apex to www). It may already be in place.
- Setup prerequisite: the **apex must be configured in Launch first**, then the **apex→www redirect** is set in the UI — see [apex→www redirect requires apex configured](launch-apex-to-www-redirect-requires-apex-configured-custom-hostnames.md).

## Status

- **Resolved.** Apex→www canonical redirect is **supported and was already working** (`wedgewood.com` → `www.wedgewood.com`, confirmed in network tab). Checked back with the customer for any additional requirements.
