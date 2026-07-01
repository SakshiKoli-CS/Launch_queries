QUESTION

Customer **Assurant** (via partner **Concord**, SF case **00046884**, **P0** go-live blocker) is cutting `www.assurant.com` over to Launch. They changed the existing **A record to a CNAME → `assurance-production.azcontentstackapps.com`** per the go-live docs, and hit **Cloudflare Error 1014**. Their domain is **on Cloudflare (proxied/"orange")**, and the Launch target is **also on Cloudflare** — Cloudflare **prohibits a CNAME pointing across two different Cloudflare accounts when the record is proxied** (ref: Cloudflare Error 1014 docs). They want to **cut over today with no downtime** and **manage the apex + apex→www redirect themselves**, pointing only `www` at Launch.

_Keywords: Cloudflare error 1014, CNAME cross account prohibited, orange to orange O2O, DNS only grey cloud unproxied, www CNAME to azcontentstackapps.com, customer domain on Cloudflare pointing to Launch on Cloudflare, cutover P0 no downtime, cert validation _acme-challenge dcv.cloudflare.com CNAME, hostname validation _cf-custom-hostname TXT, apex managed by customer point only www to Launch, zone hold release orange-to-orange._

ANSWER

## Root cause — Cloudflare Error 1014 = a PROXIED CNAME can't point across two Cloudflare accounts

- The customer's zone is on **Cloudflare with the record proxied (orange cloud)**. Launch's `*.azcontentstackapps.com` target is **also behind Cloudflare** (different account). Cloudflare **blocks a proxied CNAME from one account pointing to a hostname on another account** → **Error 1014**.

## Fix — set the `www` record to "DNS only" (grey cloud), not proxied

- **The `www` CNAME must be "DNS only" (grey cloud), NOT proxied (orange).** With the record **unproxied**, Cloudflare doesn't try to resolve it as a cross-account proxied CNAME, and Error 1014 goes away — traffic flows through to Launch (which provides the CDN/proxy itself).
- This is the standard **Cloudflare→Launch ("orange-to-orange" / O2O)** setup — see the **Go Live guide → "Cloudflare orange-to-orange (O2O)"** section. In some cross-account cases a **zone hold release** on the target side is also required.
- If it still misbehaves, try **deleting and re-adding the custom subdomain** on the Launch side, and confirm the subdomain is added **both** in the customer's **DNS (as a CNAME)** and in the **Launch UI**.

## The cert/hostname validation records still apply (do BEFORE cutover)

Before cutting over, the customer must add the validation records so certs provision (Launch manages + renews them):
```
Cert (DCV):       _acme-challenge.www.assurant.com   CNAME  www.assurant.com.<hash>.dcv.cloudflare.com
Hostname validate: _cf-custom-hostname.www.assurant.com TXT  <uuid>
```
Once **cert + hostname are validated**, they're good to go live on Launch.

## Apex handled by the customer

- Per the customer's preference, **they manage the apex (`assurant.com`) and the apex→www redirect themselves** — only the **`www` subdomain points to Launch**. (The apex→www redirect was also set up in **Fastly** on the Launch side as a fallback, but they don't need the apex configured on Launch.)

## Generalized guidance (for future similar queries)

- **Cloudflare Error 1014 on a Launch cutover** = a **proxied (orange) CNAME pointing across Cloudflare accounts**. **Fix: set the `www` record to "DNS only" (grey cloud).** This is the **Cloudflare→Launch orange-to-orange (O2O)** case in the Go Live guide; may also need a **zone hold release** on the target.
- **Customer-on-Cloudflare + Launch-on-Cloudflare** is the trigger — always check the record's **proxy status** first when a Cloudflare-hosted customer hits errors pointing `www` at `*.azcontentstackapps.com`.
- Still **add the DCV CNAME (`_acme-challenge…dcv.cloudflare.com`) + hostname TXT (`_cf-custom-hostname…`) and validate certs BEFORE cutover** — the 1014/proxy fix is separate from cert provisioning.
- **Process tell (recurring):** the Launch team **had no invite to this go-live** — customers must **schedule go-live calls in advance** (manual cutover steps involved). See related.
- Related: [multi-domain go-live full process (www Cloudflare / apex Fastly)](../custom-domains-ssl/launch-multi-domain-golive-apex-fastly-www-cloudflare-full-cutover-process-dns-records-bfs.md), [go-live support request in advance + standard steps](../custom-domains-ssl/launch-golive-support-request-in-advance-standard-steps-schedule-weekday-ist-pods.md), [IP-restrict default domain / Cloudflare custom CDN](ip-restrict-default-domain-cloudflare-custom-cdn.md), [Imperva WAF Cloudflare error 1000 host header](launch-imperva-waf-cloudflare-error-1000-host-header.md), [cert pending → DCV needs CNAME not TXT](../custom-domains-ssl/launch-cert-pending-dcv-needs-cname-not-txt-domain-limit-golive.md).

## Status

- **Resolved — cut over successfully** (marathon P0 troubleshooting call with Assurant + Concord + Launch). Cause = **Cloudflare Error 1014**: a **proxied cross-account CNAME**. Fix = set the **`www` record to "DNS only" (grey cloud)** (Cloudflare→Launch **orange-to-orange / O2O**). Cert/hostname validated via **`_acme-challenge…dcv.cloudflare.com` CNAME + `_cf-custom-hostname…` TXT** before cutover; **apex + apex→www managed by the customer** (only `www` pointed to Launch; apex→www also set in Fastly as fallback). Recurring process note: **Launch had no go-live invite** — schedule cutover calls in advance.
