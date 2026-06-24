QUESTION

Customer **Service Foods** (SF case **00057534**, AWS NA, urgent — trying to go live) reported their custom-domain project's **certificate stuck in "Pending" for 30+ minutes**, despite having added **a TXT record and a CNAME**.

- Org `blt23c9f8e216f35d7b`, Domain `servicefoods.co.nz`.

_Keywords: certificate pending, cert not provisioning, custom domain pending, DCV CNAME not TXT, _acme-challenge CNAME, hostname validation TXT, cert validation CNAME, domain limit Small plan, partially configured domain, can't edit delete domain, go-live support, apex and www._

ANSWER

## Primary cause — DCV cert validation needs a CNAME, NOT a TXT record

- Launch uses **two different records** for a custom domain:
  - **Hostname validation → TXT record**
  - **Certificate (DCV) provisioning → CNAME record**
- The customer had added a **TXT** record where the **`_acme-challenge` CNAME** was required for cert provisioning. With the wrong record type, the **certificate stays Pending**.
- **Correct DCV record (example shape):**
  ```
  _acme-challenge.servicefoods.co.nz  CNAME  servicefoods.co.nz.<id>.dcv.cloudflare.com
  ```
  Replace the TXT on `_acme-challenge` with the **CNAME** value Launch provides.
- (Docs were flagged for clarification — the two record types, **CNAME for cert + TXT for hostname**, weren't clearly distinguished.)

## Secondary cause — partially-configured domain blocking edit/delete (+ domain limit)

- The domain was in a **partially-configured state**, which **prevented the user from editing or deleting** the domain entry. This may have been related to **insufficient domain limits**.
- Fix: support **removed the domain entry from the backend** on a call, which **unblocked** the user to re-add it correctly.

## Plan domain limit — 1 on "Small"; a site usually needs 2 (apex + www)

- The **Small** Launch plan includes **1 domain** (also 1 repo, 3 envs, 250 hrs compute). A typical site needs **two**: the **apex** (`servicefoods.co.nz`) **and `www`** (`www.servicefoods.co.nz`).
- The **domain limit was increased from 1 → 2** to accommodate apex + www. (Plan upgrades/costs are handled by the account team, not engineering.)

## Go-live support — request in advance

- Engineers can join a **go-live bridge**, but it must be **requested/scheduled in advance** (via #support-launch or a support request), per the **Go Live Guide → "cutover with online support"**. Raising the go-live request and **domain requirements early** would have let the limit be adjusted proactively and avoided the blockage.

## Resolution

- Replaced TXT→CNAME for DCV, cleared the partially-configured entry, raised domain limit to 2 → cert validated, customer **went live** successfully.

## Generalized guidance (for future similar queries)

- **"Certificate stuck Pending even though we added the records"** → check the **record TYPE**: cert/DCV needs a **CNAME** (`_acme-challenge... CNAME ...dcv.cloudflare.com`), hostname validation needs a **TXT**. A TXT in place of the DCV CNAME keeps the cert Pending. (Try **Revalidate** in Domains → Actions after fixing.) Related: [pending validation / DCV entry](launch-custom-domain-pending-validation-dcv-records.md).
- **"Can't edit/delete a domain / partially-configured"** → may need support to **remove the stale entry from the backend**; can be tied to **domain-limit** constraints.
- **Domain limits are per plan** (Small = 1); a site usually needs **apex + www = 2** — surface this early so the limit can be raised before go-live.
- **Going live?** Request **on-call go-live support in advance** (Go Live Guide) and state domain requirements up front.

## Status

- **Resolved** — DCV CNAME corrected, backend entry cleared, domain limit raised to 2; customer **went live**.
