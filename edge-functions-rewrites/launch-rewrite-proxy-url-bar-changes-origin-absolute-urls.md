QUESTION

While configuring **rewrite rules** for the BAYFC site (proxying content from `sbayfc.wpengine.com`), the rewrites work and pages render correctly — but the **browser URL bar changes to the WP Engine domain** instead of staying on the customer's domain.

Example: clicking a header item ("Single Match Tickets") correctly proxies to `sbayfc.wpengine.com/tickets/single-match-tickets/` and content loads, **but the URL becomes `sbayfc.wpengine.com`** rather than remaining on the Launch/custom domain.

_Keywords: rewrite proxy URL bar changes, browser redirects to origin domain, WP Engine domain showing, absolute URLs from origin, WordPress site URL, canonical URL wrong domain, reverse proxy URL leak, links point to backend, rewrite vs redirect._

ANSWER

## Cause — the origin (WordPress) emits absolute URLs with its own domain

- When a request comes via Launch, it's **proxied to WordPress** in the background. WordPress generates the page and returns the **full response — including links, redirects, and metadata.**
- WordPress is configured with **`sbayfc.wpengine.com` as its site URL**, so it builds **all links (menu items, buttons), redirects, and metadata (e.g. canonical URLs) using that domain** — the responses **already contain the WP Engine URL**.
- **Launch does not modify the response** — it returns WordPress's output as-is. So when a link is clicked or a redirect fires, the browser **follows the absolute WP Engine URL** and the address bar changes.
- **This is not a Launch issue** — it's how the **origin (WordPress) is configured to generate absolute URLs**; the rewrite proxies content but can't know to rewrite hardcoded absolute links in the body.

## Direction of the fix

- The clean fix is **origin-side**: configure WordPress to **generate URLs using the public (Launch/custom) domain** or **relative URLs** — e.g. set WP **Site Address / Home (`WP_SITEURL` / `WP_HOME`)** to the public domain, use relative links, and fix **canonical/redirect** domains (a domain search-replace in WP).
- A **rewrite (proxy)** keeps the URL on your domain only if the **response body's links are relative or already your domain**; **absolute origin URLs in the HTML/redirects** will still navigate users to the origin. (A redirect, by contrast, is *expected* to change the URL — confirm you want a rewrite, not a redirect.)
- Launch-side body-rewriting of absolute URLs is non-trivial; if pursued, it would be via an **Edge Function** rewriting response links — but the **recommended path is fixing the origin's URL generation.**

## Generalized guidance (for future similar queries)

- **"Rewrite/proxy works but the URL bar switches to the backend/origin domain"** → the **origin is emitting absolute URLs** (links/redirects/canonicals) with its own hostname; Launch forwards them unchanged. **Not a Launch bug.**
- **Fix at the origin:** set the origin's site/home URL to the **public domain** and/or use **relative URLs**; correct canonical + redirect domains.
- **Rewrite vs redirect:** a rewrite should keep your domain only if body links are relative/your-domain; a redirect changes the URL by design.
- Related (proxying a legacy backend through Launch): [launch-phased-migration-rewrites-legacy-backend.md](launch-phased-migration-rewrites-legacy-backend.md).

## Status

- Explained the cause (origin-generated absolute URLs, Launch passes through unchanged). Both sides to look for workarounds; Launch investigating any edge-side option, customer reviewing WordPress URL generation. Reconnect planned.
