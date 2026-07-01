QUESTION

Customer **CNE (Cherokee Nation Entertainment)** — Production Launch Setup for an upcoming **go-live**. The domain/hostname allocation needs to be **increased** to add more custom subdomains (e.g. `www.onestarrewards.com`, `www.cherokeecasino.com`, `www.hardrockcasinotulsa.com`). Two asks: **(1)** increase the limit quickly given the imminent go-live, and **(2)** provide the customer the **DNS records to add** (hostname pre-validation + domain-control-validation) so the new subdomains can be validated and issued certs.

_Keywords: domain limit increase for go-live, superadmin support request increase custom domain limit, add subdomain to environment custom domains, provide CNAME and TXT records to customer, _cf-custom-hostname TXT hostname pre-validation, _acme-challenge CNAME domain control validation dcv cloudflare, dcv.cloudflare.com validation value, temporary limit increase review contract cost, add domain then get validation records._

ANSWER

## Flow — increase the limit, add the subdomain, then hand over DCV records per domain

1. **Increase the domain/hostname limit** (done via the internal **Superadmin / #support-superadmin** request). For an **imminent go-live**, the limit can be **increased immediately to unblock** the customer; the **contractual alignment (and any cost implications) is reviewed separately/after** — a **temporary increase to unblock is not the final contract decision** and is handled once the responsible owner (e.g. account/Dean) is back.
2. **Tell the customer to add each new subdomain** (e.g. `www.onestarrewards.com`) to the **relevant Launch environment**, the same way other custom domains were added.
3. **Once the domain is added, Launch generates two DNS records per subdomain** — hand them to the customer to create at their DNS provider:
   - **Hostname pre-validation** — a **TXT** record:
     ```
     Name:  _cf-custom-hostname.<subdomain>
     Value: <uuid>            e.g. 6f993160-5853-4df5-92a1-9d010d9cc73f
     ```
   - **Domain Control Validation (DCV)** — a **CNAME** record (proves control so the cert issues):
     ```
     Name:  _acme-challenge.<subdomain>
     Value: <subdomain>.<hash>.dcv.cloudflare.com
            e.g. www.cherokeecasino.com.d181f5c75a39ca6c.dcv.cloudflare.com
     ```
   - **Each subdomain gets its OWN pair** of records. Example set:
     ```
     www.cherokeecasino.com
       _cf-custom-hostname.www.cherokeecasino.com   TXT    6f993160-5853-4df5-92a1-9d010d9cc73f
       _acme-challenge.www.cherokeecasino.com       CNAME  www.cherokeecasino.com.d181f5c75a39ca6c.dcv.cloudflare.com
     www.hardrockcasinotulsa.com
       _cf-custom-hostname.www.hardrockcasinotulsa.com  TXT    398ceb5c-b443-4701-87eb-df516ecd4b92
       _acme-challenge.www.hardrockcasinotulsa.com      CNAME  www.hardrockcasinotulsa.com.d181f5c75a39ca6c.dcv.cloudflare.com
     ```
4. **The validation records only exist after the domain is added** — so for a subdomain not yet added (e.g. `www.onestarrewards.com`), the customer **adds it first**, then Launch **follows up with that subdomain's TXT + CNAME**.

## Note on who communicates the technical details

- Whoever is already in **direct contact with the customer's technical team** should typically **relay the DNS records directly** (rather than routing through a business contact), mentioning leadership is aware if helpful. Time-zone differences are a valid reason to hand off the customer update — but the internal request must actually be **picked up** for the update to happen.

## Generalized guidance (for future similar queries)

- **Domain limit blocking a go-live** → the limit can be **increased immediately to unblock**; **contract/cost alignment is reviewed separately afterward** (a temporary unblock ≠ the final entitlement decision). The domain limit is **global across environments**, not per-environment.
- **Per new custom subdomain, the customer adds two DNS records** at their provider:
  - **`_cf-custom-hostname.<subdomain>` TXT** = hostname pre-validation.
  - **`_acme-challenge.<subdomain>` CNAME → `<subdomain>.<hash>.dcv.cloudflare.com`** = DCV (issues the cert). **DCV is a CNAME, not a TXT.**
- **Sequence matters:** the subdomain must be **added to the Launch environment first**; the **validation records are generated per-domain after it's added**. Adding these DNS records **has no impact on the existing/live setup**.
- Related: [cert pending → DCV needs CNAME not TXT](launch-cert-pending-dcv-needs-cname-not-txt-domain-limit-golive.md), [add SAN domains / DCV CNAME 7-day / 100-domain limit](launch-add-san-domains-managed-cert-100-limit-format-renewal-cname-dcv-7day.md), [custom domain pending validation DCV records](launch-custom-domain-pending-validation-dcv-records.md), [cutover cert + `_cf-custom-hostname` TXT per subdomain](launch-cutover-lets-encrypt-cert-hostname-validation-txt-apex-vs-www-jax.md), [www→apex flip + entitlement increase](launch-www-to-apex-canonical-flip-entitlement-increase-outage-redirect-recovery.md).

## Status

- **Resolved / unblocked.** Domain limit was **increased immediately** to unblock the go-live (via Superadmin request); **contract/cost alignment to be reviewed separately** once the account owner returned. Customer was told to **add each new subdomain to the relevant environment**, and was given the **per-subdomain validation records** — **`_cf-custom-hostname.<subdomain>` TXT** (hostname pre-validation) + **`_acme-challenge.<subdomain>` CNAME → `…dcv.cloudflare.com`** (DCV) — with Launch to **follow up with records for any subdomain added afterward** (e.g. `www.onestarrewards.com`).
