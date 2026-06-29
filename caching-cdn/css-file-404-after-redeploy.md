QUESTION

After a code release that changes CSS, Next.js generates a new hashed filename (`entry-client-<hash>.css`) and deletes the old one. The customer (Next Retail Limited; org `blte250adf4396ee460`, Azure EU; SF-00054163) reported that pages still holding a reference to the old CSS filename break, because the file is no longer accessible. They requested that Launch retain the old CSS file for approximately 24 hours to allow cached pages time to pick up the new version.

The core question: can Launch hold onto deleted build assets across deployments, and if not, what is the correct fix?

ANSWER

**Platform capability**

Launch **cannot retain build assets from a previous deployment** once that deployment has been removed or redeployed. Static files (including hashed CSS bundles) are tied to the deployment they were built in and are not carried forward.

Requesting a 24-hour asset retention grace period is not a supported platform feature.

**Root cause investigation — why is old HTML still being served?**

Under normal circumstances, when a site is redeployed, both the HTML pages and their referenced assets should be purged from the CDN simultaneously. If old pages are still being served after a deploy and referencing deleted CSS, the likely explanation is one of the following:

| Scenario | Mechanism | Symptom |
|---|---|---|
| `max-age` on pages (not `s-maxage`) | Browser caches old HTML beyond CDN TTL; CDN purges on deploy but user's browser still serves stale HTML referencing old CSS hash | CSS 404 only for some users; clears on hard refresh |
| External / additional CDN in front of Launch | Launch's CDN purge does not propagate to the upstream CDN layer; the upstream continues to serve old HTML | CSS 404 at CDN level; affects all users routed through that layer |

**Suggested checks**

- Inspect the `Cache-Control` header on page responses: confirm pages use `s-maxage` (CDN-controlled) rather than `max-age` (browser-controlled). If `max-age` is set, the browser will cache the old HTML and continue requesting the deleted CSS hash even after the CDN is purged.
- Confirm whether the customer has a CDN (Akamai, Cloudflare Enterprise, Fastly, etc.) sitting in front of Launch. If so, cache invalidation must be triggered separately on that layer — Launch's purge does not propagate upstream. See @caching-cdn/launch-cloudflare-apo-cache-purge-multi-cdn-invalidation.md.
- If using hooks for post-deploy cache purge, verify the hook is actually firing and the external CDN cache is being cleared on deploy.

**Recommended fix (app-side)**

If the customer cannot immediately fix the caching headers, a stable workaround is to **use a consistent (non-hashed) CSS filename** for the entry client stylesheet, or configure the bundler to reuse the same output filename. This way, old HTML references continue to resolve after a deploy, since the URL does not change. The trade-off is that browsers may serve a stale CSS file until their local cache expires — mitigated by setting a short `max-age` on the CSS file or deploying a cache-busting query string instead of a hash in the filename.

**Resolution status**

Open — customer acknowledged and is looking into their caching options. No platform-side changes planned; fix is app/infrastructure-side.
