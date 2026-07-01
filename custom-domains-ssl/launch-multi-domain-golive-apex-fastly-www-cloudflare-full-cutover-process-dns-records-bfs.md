QUESTION

Customer **Bibby Financial Services (BFS)** (via SI **iO Digital**, cases incl. **00047083**) is doing a **phased multi-country go-live** onto Launch — Singapore (`bibbyfinancialservices.sg`) first, then 8 more apex domains a week later (`.com`, `.cz`, `.de`, `bibbyfactor.fr`, `.ie`, `.nl`, `.pl`, `.sk`). They repeatedly ask: **what is the purpose of the DNS records we were given, what's the full go-live process, and exactly which records/values do we add for each domain (apex + www), and when?** Also: **will renewing certs on their existing/old sites interfere with the new Launch setup?**

_Keywords: multi-domain go-live process, apex plus www cutover, CNAME TXT records purpose, DCV certificate provisioning CNAME _acme-challenge fastly-validations.com, hostname validation TXT _cf-custom-hostname, www CNAME to eu-azcontentstackapps.com, apex A record 151.101.66.137 Fastly, apex to www redirect handled by Launch Fastly, when to switch DNS on go-live day, reduce TTL before cutover, renewing old site certs no impact on new setup, phased country domains launch._

ANSWER

## The two layers: **www subdomain** (Cloudflare) vs **apex** (Fastly) — different records

Launch serves the **www subdomain via Cloudflare** and the **apex via Fastly**, so each domain has **two separate setups** with **different validation records**:

| | **www subdomain** (`www.example.com`) | **apex** (`example.com`) |
|---|---|---|
| Served by | Cloudflare custom hostname | Fastly |
| Cert / DCV record | validation via Cloudflare (`_cf-custom-hostname` TXT + `_acme-challenge` CNAME → `…dcv.cloudflare.com`) | **`_acme-challenge.<apex>` CNAME → `…fastly-validations.com`** |
| Go-live DNS change | **CNAME** `www.<domain>` → the Launch URL (e.g. `bibby-web-project-production.eu-azcontentstackapps.com`) | **A record** `<apex>` → **`151.101.66.137`** (Fastly) |
| Apex→www redirect | — | handled by **Launch (in Fastly)** |

## Purpose of the records (what to tell the customer)

- **CNAME records** = **TLS certificate provisioning** (DCV / domain control validation) — they prove control so the cert can be issued.
- **TXT records** = **hostname ownership validation** for the custom hostname.
- These are added **ahead of go-live** and **have no impact on the existing/live site** — they only let Launch validate and pre-provision certs so the actual cutover is fast.

## The go-live process (ordered)

1. **Reduce the TTL** on the DNS records beforehand (so the cutover propagates fast).
2. **Add the subdomain** (`www.<domain>`) in the **Launch UI** on the **correct production environment**.
3. **Customer adds the validation records** (CNAME for DCV + TXT for hostname) at their DNS provider and **notifies Launch**.
4. **Launch validates + provisions certs** (both www via Cloudflare and apex via Fastly) and **confirms**.
5. **Launch enables the apex→www redirect** (in Fastly).
6. **On go-live day only**, the customer flips the live DNS:
   - **apex A record → `151.101.66.137`**,
   - **`www` CNAME → the Launch project URL** (the value is shown in **Launch UI → Project → Environment settings → Domains** — only update this on the actual go-live day).
7. **Launch is on the go-live call** to walk through the DNS switch and confirm a clean cutover.

## Renewing certs on the OLD sites — no impact

- **Renewing/replacing certificates on the customer's existing (old-host) sites does NOT interfere** with the new Launch cert provisioning. The Launch certs are provisioned independently (Cloudflare for www, Fastly for apex); the customer can keep the old certs valid until every country site is cut over.

## Process/ownership notes (repeated confusion in this thread)

- **Clarify apex-redirect ownership explicitly.** BFS said on a call they'd handle apex→www themselves, then later said they needed Launch to do it. **Confirm in writing who owns the apex→www redirect** — if Launch, it's set up in **Fastly** and the customer just points the apex **A record** at `151.101.66.137`.
- **Route go-live/DNS asks through the support case**, not ad-hoc to engineering — engineering-owned email threads get missed when people are OOO. Have a CSE/support owner drive the customer comms.

## Generalized guidance (for future similar queries)

- **"What are these records for / what's the go-live process?"** → **CNAME = cert/DCV provisioning, TXT = hostname validation**; add them **early (no impact on live site)**, then **flip apex A record + www CNAME only on go-live day**.
- **www is on Cloudflare, apex is on Fastly** — they need **different DCV records** (`…dcv.cloudflare.com` CNAME for www; **`_acme-challenge.<apex>` CNAME → `…fastly-validations.com`** for apex) and **different go-live records** (www **CNAME** to the Launch URL; apex **A → `151.101.66.137`**).
- **Apex→www redirect is handled by Launch in Fastly** when the customer points the apex A record at the Fastly IP — but **confirm ownership explicitly** (customers often flip-flop on whether they or Launch handle it).
- For **phased multi-domain go-lives**, provision **per-domain `_acme-challenge` CNAMEs** in batches ahead of each wave; **all apex domains point to the same Fastly IP (`151.101.66.137`)**.
- **Renewing old-site certs doesn't affect the new Launch provisioning** — safe to keep the old site secure through the migration.
- Related: [cutover cert + `_cf-custom-hostname` TXT per subdomain (Jax)](launch-cutover-lets-encrypt-cert-hostname-validation-txt-apex-vs-www-jax.md), [apex cert renewal / move apex to Launch (Fastly redirect)](launch-apex-cert-renewal-expired-redirect-move-apex-to-launch.md), [domain-limit increase + per-subdomain DCV/hostname records](launch-domain-limit-increase-golive-provide-dcv-txt-cname-add-subdomain-cne.md), [apex→www redirect requires apex configured](launch-apex-to-www-redirect-requires-apex-configured-custom-hostnames.md), [cert pending → DCV needs CNAME not TXT](launch-cert-pending-dcv-needs-cname-not-txt-domain-limit-golive.md).

## Status

- **Resolved — Singapore + 8 country apex domains launched smoothly.** Per domain: **apex set up in Fastly** (cert via **`_acme-challenge.<apex>` CNAME → `…fastly-validations.com`**, apex→www redirect in Fastly), **www via Cloudflare custom hostname**. Go-live flip = **apex A record → `151.101.66.137`** + **www CNAME → Launch project URL** (value from **Launch UI → Environment settings → Domains**), done **on go-live day** on a support call. Confirmed **renewing old-site certs had no impact** on the new setup. Key friction was **process/ownership clarity** (who handles apex→www; routing asks through support vs engineering), not a technical fault.
