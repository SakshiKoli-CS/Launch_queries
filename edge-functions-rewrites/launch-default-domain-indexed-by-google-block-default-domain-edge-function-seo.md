QUESTION

Customer **UVA Health** (SF case **00050181**) finds that **Google is indexing the `*.azcontentstackapps.com` default domain** instead of the custom domain — e.g. `https://uvahealth.azcontentstackapps.com/providers/…` appears in search results instead of `https://uvahealth.com/providers/…`. The `.azcontentstackapps.com` domain was **auto-generated** when the environment was created. They already use Edge Functions (www→non-www). They ask:

1. Is it **safe to add an Edge Function redirecting all `*.azcontentstackapps.com` → `uvahealth.com`**?
2. Could that **cause a redirect loop** (since the production env's **default domain IS `*.azcontentstackapps.com`**)? — they previously hit redirect loops post-launch.
3. **Best practice** to keep internal URLs out of Google (robots.txt / redirects / HTTP headers)?

_Keywords: azcontentstackapps.com indexed by Google, default domain in search results, SEO internal URL indexing, block default domain edge function, redirect default domain to custom domain, redirect loop concern default domain, robots.txt X-Robots-Tag noindex, prevent crawlers default domain._

ANSWER

## Recommended fix — Edge Function to BLOCK direct requests to the default domain

- Set up an **Edge Function that blocks all direct requests reaching the Contentstack default domain (`*.azcontentstackapps.com`)** and **only allows requests on the custom domain** (`uvahealth.com`).
- This **prevents crawlers from crawling the default domain** and stops the `*.azcontentstackapps.com` links from appearing in Google search results.
- **Blocking (vs redirecting) is the cleaner approach** for the SEO goal and avoids the redirect-loop risk the customer worried about.
- Example implementation: **`contentstack-launch-examples/launch-edge-default-domain-blocking`** (see the domain-migration / edge entries).

## On the redirect-loop concern

- The customer's worry is valid: because the **production environment's default domain IS `*.azcontentstackapps.com`**, a naive "redirect everything on the default domain → custom domain" can misfire. **Blocking direct default-domain requests (allow-list the custom domain) sidesteps the loop** — you're not bouncing requests in a cycle, just refusing/blocking the default host.

## Additional SEO hardening

- Combine with **`robots.txt`** and/or **`X-Robots-Tag` / `noindex`** headers for extra assurance that the internal/default domain isn't indexed.

## Generalized guidance (for future similar queries)

- **"Google is indexing our `*.azcontentstackapps.com` default domain instead of the custom domain"** → the auto-generated default domain is publicly reachable and crawlable. **Fix: an Edge Function that BLOCKS direct requests to the default domain and only serves the custom domain** (`launch-edge-default-domain-blocking` example) → crawlers can't index it.
- **Prefer BLOCK over REDIRECT** for the default→custom case — avoids redirect loops (the env's default host is the same `*.azcontentstackapps.com`).
- **Harden with `robots.txt` + `X-Robots-Tag: noindex`** on the default domain.
- Related: [block default domain example / intermittent 404 entry](../logs-monitoring/launch-intermittent-404-etimedout-cda-timeout-vs-invalid-locale-422-next.md), [default-domain "Dangerous Site" noindex/robots](../custom-domains-ssl/launch-contentstackapps-dangerous-site-noindex-robots-assets-blocked.md), [domain migration redirects via edge function](launch-domain-migration-redirect-old-domains-to-subfolders-edge-function-domain-limit.md).

## Status

- **Answered.** To stop Google indexing the **`*.azcontentstackapps.com` default domain**, use an **Edge Function that blocks direct requests to the default domain and only allows the custom domain** (`launch-edge-default-domain-blocking` example) — this prevents crawling/indexing and **avoids the redirect-loop risk** (block, don't redirect). Add **robots.txt / `X-Robots-Tag: noindex`** as extra hardening.
