# Launch Support Knowledge Base

Markdown KB entries distilled from Contentstack **Launch** support conversations, organized by **feature area**. Each entry follows the QUESTION / ANSWER pattern (see `.cursor/rules/support-pattern.mdc`).

**How to use this index:** scan for the symptom or keyword that matches the query, then open the linked entry. Authoring/extraction rules and folder placement live in [`AGENTS.md`](AGENTS.md) and the support-pattern rule.

---

## custom-domains-ssl/ — custom domains, apex/A records, DCV, SSL/TLS

- [Custom domain & SSL not active; no A record for apex](custom-domains-ssl/launch-custom-domain-apex-no-a-record-ssl-not-active.md) — apex must be **created in the Launch UI first**; A record + cert are only generated after that. _Keywords: custom domain inactive, no A record, SSL not active, apex domain._
- [Custom domain stuck in "Pending Validation" / DCV records](custom-domains-ssl/launch-custom-domain-pending-validation-dcv-records.md) — verify **DCV + TXT records** at DNS provider; stuck-but-correct domains can clear on upstream (Cloudflare) delay. _Keywords: pending validation, DCV, certificate stuck, TXT record._
- [SES click tracking on a branded custom domain (HTTPS)](custom-domains-ssl/launch-ses-click-tracking-custom-domain-https-proxy.md) — `server: cloudflare` is normal; CNAME direct to SES for HTTP, **Next.js catch-all proxy route** for branded-cert HTTPS. _Keywords: AWS SES, awstrack.me, Braze, cert mismatch, TLS SNI._

## edge-functions-rewrites/ — Edge Functions, Edge URL Rewrites / launch.json

- [Edge Functions in TypeScript](edge-functions-rewrites/edge-functions-typescript.md) — Edge Functions run **JavaScript only**; author in TS, **compile to JS** into the `functions/` folder (sample `tsconfig.json` included). _Keywords: edge function, TypeScript, JavaScript only, tsconfig, build to functions folder._
- [launch.json rewrites "lost" after a failed deployment](edge-functions-rewrites/launch-json-rewrites-config-on-failed-deployment.md) — failed deploy does **not** wipe config; last successful config stays live. File must be at **project root**. _Keywords: launch.json, edge rewrites, config lost, Vercel migration._

## deployments-builds/ — deployments, build machines, build/install failures

- [Build fails with no error in log → enable L2 build machine](deployments-builds/launch-build-failed-no-error-l2-build-machine.md) — log stops mid-process = under-provisioned machine; enable `contentfly_deployment_build_machine_l2`. _Keywords: build failed no error, stuck mid-log, L2 build machine, Enterprise default tier._
- [npm E401 "Incorrect or missing password" on install](deployments-builds/launch-build-npm-e401-auth-token-npmrc.md) — private-registry **auth token** missing in build env (works locally via cached creds); fix `.npmrc` + env vars. _Keywords: npm E401, .npmrc, NPM_TOKEN, private registry, fontawesome/GitHub Packages._
- [Blank pages from `.next/cache` ENOENT on read-only FS](deployments-builds/launch-nextjs-cache-enoent-blank-pages-readonly-fs.md) — `/var/task` is read-only; relocate cache to `/tmp` via `cacheHandlers`. ENOENT ≠ ENOSPC. _Keywords: blank pages, .next/cache, ENOENT, read-only filesystem, cacheHandlers._
- [Deployment queued/failed + Live Preview not working](deployments-builds/launch-deployment-queued-failed-live-preview-not-working.md) — separate the deploy failure from Live Preview (iframe CORS/CSP); check SSO-only access for support. _Keywords: deployment queued, Live Preview SDK not initialized, CORS/CSP iframe._

## caching-cdn/ — CDN caching, invalidation, origin traffic, personalization

- [CDN caching vs personalization; no cache-key variation by cookie](caching-cdn/launch-cdn-caching-personalization-no-cache-variation.md) — Launch caches by **URL only** (no cookie/header Vary); use `no-store` for personalized SSR, CDN-cache only shared data; no `revalidateTag/Path`. _Keywords: personalization, cookie cache, no-store, s-maxage, Revalidate CDN Cache API._
- [Sustained origin traffic surge / 404s on unpublished asset](caching-cdn/launch-sustained-origin-traffic-surge-404-unpublished-asset.md) — external crawlers hitting historical URLs (not Launch caching); Tiered Cache ≠ per-edge revalidation; block/observe via Edge Function. _Keywords: traffic surge, 404 unpublished entry, tiered cache, node-fetch, 24/7 non-diurnal._

## logs-monitoring/ — Log Targets, log volume, uptime monitoring

- [Estimating Non-Production log volume](logs-monitoring/launch-nonprod-log-volume-estimate-log-targets.md) — Launch can't estimate it (internal format ≠ forwarded, short retention); measure via a **Log Target** in the observability platform. _Keywords: log volume, Log Target, OpenTelemetry, ingestion volume, retention._
- [Site down "Request URI does not exist" / redirect 404](logs-monitoring/microstrategy-site-down-uri-does-not-exist-redirect-404.md) — customer rebrand/deploy dropped a redirect mapping; not a platform fault; monitor target may need updating. _Keywords: site down, Request URI does not exist, redirect 404, monitor, rebrand._

## networking-connectivity/ — egress/outbound IPs, allowlisting, connectivity errors

- [No fixed outbound/egress IPs for allowlisting](networking-connectivity/launch-no-fixed-outbound-egress-ip-allowlist.md) — Launch has **no static egress IPs**; use auth-secured endpoint or a customer proxy/gateway with a static IP. Marketplace App IPs ≠ Launch. _Keywords: outbound IP, egress IP, allowlist, firewall, 403, static IP roadmap._
- [DELETE method with body → CF1003 / 502](networking-connectivity/launch-delete-method-body-cf1003.md) — Launch **doesn't support a payload on GET/DELETE** (RFC 7231); DELETE+body fails upstream before reaching the handler. Move identifiers to **query params**. _Keywords: CF1003, 502, DELETE with body, upstream rejected, RFC 7231._
- [Next.js OAuth authorize/state → 520](networking-connectivity/nextjs-oauth-simple-authorize-state-520.md) — **520** = origin didn't return a valid response (not a WAF block, which is 403); extra redirects + `state`/latency in the OAuth chain caused incomplete responses. _Keywords: OAuth, simple-authorize, state param, 520, redirects, slow origin._

## integrations/ — Git/GitHub, Marketplace apps, third-party

- [Single GitHub org connection; repo migration A→B](integrations/launch-github-single-org-connection-repo-migration.md) — one GitHub org connection at a time; disconnecting **does not** break live deployments; multi-org is roadmap. _Keywords: GitHub integration, single org, repo migration, disconnect, roadmap._

---

## Reference (root)

- [Brief understanding of launch.md](Brief%20understanding%20of%20launch.md) — overview of Launch.
- [AGENTS.md](AGENTS.md) / [CLAUDE.md](CLAUDE.md) — authoring instructions & folder rules.
