QUESTION

Customer **PODS** (SF case **00050365**) reports users getting **"Access Denied" (HTTP 403)** on certain URLs — specifically paths containing **"partners"** (e.g. `https://www.pods.ca/partners/caa`) — on `stage.pods.com`. The env served the **correct Contentstack SSL cert**; **no recent code changes.**

Diagnostic signals:
- **Homepage works**; only some paths 403.
- A **cURL to the affected URL returned 200 OK.**
- **Before clearing cache: CF cache status MISS; after clearing: HIT** — **clearing the cache resolved it.**
- Started in prod at a specific time; `stage.pods.com` is a CNAME.

They asked Launch to review **WAF/CDN logs** (blocked at edge? rule ID/reason?), and whether there were **recent WAF/security rule updates** or Fastly edge behavior on invalid paths/cookies.

_Keywords: 403 Access Denied certain paths, partners path blocked, cURL 200 but browser 403, clearing cache fixes, CF cache MISS then HIT, platform-wide WAF rule change, GEO restriction sanctioned countries OWASP rules added inadvertently, no per-user WAF rules, Fastly edge block, blocked at edge rule ID._

ANSWER

## Cause — a platform-wide WAF rule change accidentally added extra OWASP rules

- A **Contentstack-wide WAF rules configuration change** went through recently for **compliance** — **GEO-restriction policies** blocking access from specific sanctioned countries/regions (Russia, Belarus, Iran, Iraq, Libya, Cuba, Myanmar, Somalia, Sudan, Syria, Venezuela, Yemen, Zimbabwe, North Korea, and Ukraine's Crimea/Luhansk/Donetsk).
- **While adding those, a few extra rules — among the top OWASP rules — got added inadvertently.** Those extra rules **blocked legitimate requests on certain paths** → the 403s.
- **WAF is platform-wide on Launch** — each user can't have their own per-site WAF rule set; **maintaining individual per-user WAF rules isn't feasible** currently, so the change applied broadly.

## Why the confusing signals

- **cURL returned 200 / homepage fine, but the browser 403'd** → the WAF block keyed on request characteristics (path/headers/cookies) that differ between a clean cURL and a real browser request.
- **Clearing the cache "fixed" it** (MISS→HIT) → a **blocked 403 response had been cached** at the CDN; purging let a fresh (now-unblocked) response cache.

## Fix

- The **inadvertently-added extra WAF policies were removed.** **Only the GEO-location WAF rule remains.**
- Launch noted **learnings to prevent recurrence**, and may **reverse the GEO-location WAF policies** later (will notify if so).

## Generalized guidance (for future similar queries)

- **Sudden 403 "Access Denied" on specific paths, no code change, cURL 200 but browser 403, clearing cache fixes it** → suspect a **WAF rule blocking legitimate requests** (and the **403 got cached**). On Launch, **WAF is platform-wide** — a recent **platform WAF change** can cause this across sites; **escalate to check recent WAF/OWASP rule updates.**
- **Launch WAF is standard/platform-wide — no per-user rule customization.** Compliance/GEO changes apply broadly; an over-broad OWASP addition can block valid traffic.
- **Clearing the CDN cache** clears a **cached 403** once the rule is fixed — but the real fix is removing/correcting the WAF rule.
- Related: [programmatic 403 → Cloudflare WAF/Bot/Browser Integrity Check](launch-403-programmatic-access-cloudflare-waf-bot-browser-integrity-check.md), [all-URLs-redirect + Launch WAF model (no per-user rules)](../edge-functions-rewrites/launch-all-urls-redirect-to-homepage-nextjs-middleware-accept-header-waf-model.md), [Imperva WAF / Error 1000](launch-imperva-waf-cloudflare-error-1000-host-header.md).

## Status

- **Resolved.** Cause: a **platform-wide compliance WAF change** (GEO/sanctioned-country restrictions) where **extra top-OWASP rules were added inadvertently**, blocking valid paths (403, and the 403 got cached → cache clear masked it). **Extra rules removed; only the GEO-location rule remains.** WAF is platform-wide (no per-user rules); recurrence-prevention learnings noted; GEO policy may be reversed later.
