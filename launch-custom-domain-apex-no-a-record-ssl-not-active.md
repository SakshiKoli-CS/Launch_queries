QUESTION

Customer **Cherokee Nation Entertainment** (SF case **00059377**) followed the Launch custom domain setup instructions, but their **custom domain and SSL certificate were still not showing as Active**. They also reported they **did not receive an A record for their apex domain** and wanted to know when the domain/SSL would activate.

- **Domain:** apex `statusplusrewards.com` + subdomain `www.statusplusrewards.com`
- **Organization ID:** `blt24b714f15302a9f9`
- **Region:** AWS North America

_Keywords: custom domain not active, SSL certificate not issued / not active, apex domain, no A record, A record missing, CNAME, DNS records, hostname status, certificate status, domain setup, delete and re-add domain, statusplusrewards.com._

ANSWER

## Root cause

The **apex domain had not been created in the Launch UI yet.** In Launch:

- The **A record is generated only after you create and configure the apex domain** as a custom domain in the Launch UI. No apex domain created → **no A record is issued.**
- The **SSL certificate is issued only after** the domain is successfully created in the Launch UI **and** the correct DNS records are added at the DNS provider. So a "missing certificate" is downstream of the apex/domain not being fully set up.

The customer had created an A record (reusing the IP of their two previous sites) and the CNAMEs, but **because the apex domain object didn't exist in Launch, the setup couldn't complete** — hence no Launch-provided A record and no certificate.

## Correct setup flow

1. In the **Launch UI**, create a custom domain for the **apex** `statusplusrewards.com` (and the `www` subdomain).
2. Launch then **generates the required DNS values** (A record for apex, CNAME for subdomain as applicable).
3. Add **those Launch-provided DNS records** at your DNS provider — do not assume reusing an old site's IP is correct; use the values Launch generates.
4. After **DNS propagation**, the **domain and SSL certificate become active automatically.**

**Docs:** Custom Domain setup, and "Adding Apex Domain" (Launch developer docs).

## Resolution

- The customer **deleted the existing custom domain and set it up again**. On reconfiguration, the **Apex domain setup option appeared**, they configured it successfully, and confirmed **everything is now working** (both `statusplusrewards.com` and `www.statusplusrewards.com` showed **Active Certificate Status** and **Active Hostname Status**, with the apex correctly **redirecting to `www`**).
- Note: from the Launch side, status had eventually shown Active and a delete/re-add was **not strictly required** — but the re-add is what surfaced the **Apex domain setup option** for the customer and got them unblocked.

## Generalized guidance (for future similar queries)

- **"No A record for my apex domain"** → The apex must be **created as a custom domain in the Launch UI first**; the A record is only generated after that. Launch does not pre-issue an A record.
- **"SSL certificate not active / missing"** → Certificate is issued **only after** the domain is created in Launch **and** DNS records are correctly added and propagated. Check domain creation + DNS first before treating it as a cert bug.
- **"I added an A record / CNAME but domain still inactive"** → Verify they used the **Launch-generated DNS values**, not reused IPs from another site, and that the apex domain object exists in Launch.
- **Hostname Status + Certificate Status both Active** in Launch = setup is correct on the platform side; remaining issues are usually DNS propagation or a missing apex domain object.
- If the apex setup option isn't visible, **deleting and re-adding the custom domain** can surface the apex configuration step.
