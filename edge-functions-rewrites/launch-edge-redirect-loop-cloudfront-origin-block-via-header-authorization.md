QUESTION

Customer **Rapid7** (SF case **00047345**) has a **`[proxy].edge.js`** that redirects the default domains to their real domains (to **hide the publicly-accessible `*.contentstackapps.com` URLs** for security):
```
rapid7-website.contentstackapps.com          -> https://www.rapid7.com
rapid7-website-development.contentstackapps.com -> https://dev.rapid7.com
rapid7-website-staging.contentstackapps.com     -> https://staging.rapid7.com
```
Two problems, in order:
1. **The Edge Function deployment fails** (across all environments, so not env-specific in the usual sense).
2. After the deploy is fixed, the **redirect causes an infinite loop** — because their **AWS CloudFront distribution uses the `*.contentstackapps.com` URL as its ORIGIN**: a user hits `www.rapid7.com` → CloudFront fetches from `rapid7-website.contentstackapps.com` → the edge function redirects back to `www.rapid7.com` → loop.

_Keywords: edge function deployment fails, redirect default domain to real domain, hide contentstackapps URLs security, CloudFront origin is contentstackapps URL, redirect loop own CDN origin, block direct access header authorization, custom identifier header CloudFront, allow only requests with header block others, SEO bot indexing default domain, env var enclosing quotes deploy fail._

ANSWER

## Part 1 — Edge Function deploy failure = malformed env vars (enclosing quotes)

- The deploy failure was **NOT the edge code** (the redirect snippet was correct). It was **malformed environment variables** — specifically **enclosing quotes** on:
  - `BROWSERSTACK_USERNAME`, `BROWSERSTACK_ACCESS_KEY`, `NEXT_PUBLIC_GTM_SCRIPT_URL`
- **Removing the enclosing quotes → deploy succeeded.** (General: also check **extra spaces / escaped `\n`**; private keys need real newlines, no quotes, entered in **Form Mode**. Diagnosing needed **read-only collaborator access** to the project to inspect the env vars.)

## Part 2 — Redirect causes a loop because CloudFront's ORIGIN is the default domain

- **Don't REDIRECT** the default domain to the real domain when **your own CDN (CloudFront) uses the `*.contentstackapps.com` URL as its origin** → you get an **infinite loop** (real domain → CloudFront → origin default-domain → edge redirects to real domain → …).
- The actual goal is to **hide / prevent public access + SEO indexing of the `*.contentstackapps.com` URLs** — solved by **blocking, not redirecting**, with a **shared-secret header between CloudFront and the edge function:**
  1. **CloudFront attaches a specific identifier header** to all requests it sends to `rapid7-website.contentstackapps.com` (acts as an authorization token).
  2. The **Launch Edge Function allows only requests that include that header** (i.e. legitimate traffic proxied via CloudFront).
  3. **Block all requests without the header** → direct hits and **SEO bot crawls of the default domain are blocked**, so they don't get indexed/served — **without redirecting** (no loop).

## Generalized guidance (for future similar queries)

- **Edge Function deploy fails (all envs), code looks correct** → **audit env vars for enclosing quotes / spaces / escaped `\n`** (private keys: real newlines, no quotes, Form Mode). Need **read-only collaborator access** to inspect. (Same class as other edge-deploy env-var failures.)
- **"Hide the public `*.contentstackapps.com` default domain" when your CDN uses it as ORIGIN** → **do NOT redirect** default → real domain (infinite loop). Instead **block direct/unauthorized access at the edge** using a **shared identifier header set by your CDN (CloudFront)** — edge function **allows only header-bearing requests, blocks the rest** (kills direct access + SEO bot crawling; no loop).
- This header-auth-block is the robust variant of "block the default domain" when a **CDN-over-Launch origin** relationship exists.
- Related: [edge function deploy fails one env → malformed env var quotes](launch-edge-function-deploy-fails-one-env-malformed-env-var-quotes.md), [block default domain to stop Google indexing](launch-default-domain-indexed-by-google-block-default-domain-edge-function-seo.md), [all-URLs-redirect / Launch WAF model](launch-all-urls-redirect-to-homepage-nextjs-middleware-accept-header-waf-model.md).

## Status

- **Resolved (both parts).** (1) Edge deploy failure = **env vars with enclosing quotes** (`BROWSERSTACK_*`, `NEXT_PUBLIC_GTM_SCRIPT_URL`) → removed quotes → deploys fine. (2) Redirecting default→real domain **looped because CloudFront's origin is the default domain**; fix is to **block (not redirect)** — **CloudFront sets an identifier header; the edge function allows only header-bearing requests and blocks the rest**, hiding the default URLs from direct/bot access without a loop.
