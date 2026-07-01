QUESTION

Customer **First Interstate Bank** (`firstinterstatebank.com`, a **`.bank`-adjacent financial institution**), freshly migrated to Launch, reports that **older browsers (notably Safari & Firefox on old Macs) show the site as "not secure" / "not trusted"** — end users (esp. an older, less-technical demographic) think their **bank login is a scam site**. Newer browsers are fine. **This is a major issue (not an outage).** The bank's own engineers diagnosed it: the leaf cert chains up to the **Sectigo R46** intermediate, but **old Macs don't have that root in their cert store**, so the chain doesn't validate. They **had the same issue on their previous host** and fixed it by presenting the **cross-signed root** in the chain. Their ask: **include the "Sectigo Public Server Authentication Root R46 x USERTrust RSA CA" cross-signed cert in the chain** Launch/Cloudflare presents.

_Keywords: SSL not trusted old browsers, site not secure warning bank, Safari Firefox old Mac cert error, missing intermediate root in trust store, Sectigo R46 not trusted, cross-signed root USERTrust RSA CA, add cross-chain cert to certificate chain, SSL Labs A+ extra download, HSTS enable, cross-signing problem, older demographic scam site, financial institution escalation._

ANSWER

## Root cause — the cert chain didn't include the cross-signed root, so old trust stores can't validate it

- The leaf cert chains to **Sectigo R46**, which **newer browsers trust natively** — but **older browsers/OSes (old macOS Safari & Firefox) don't have Sectigo R46 in their trust store**, so the chain **fails to validate** → "not secure" warning.
- The fix (the bank's own diagnosis, standard cross-signing practice): **present the cross-signed root** — **"Sectigo Public Server Authentication Root R46 x USERTrust RSA CA"** — in the chain. USERTrust RSA CA is an **older, widely-distributed root** present even in old trust stores, so cross-signing R46 to it lets old clients build a valid chain.
- **SSL Labs** flags this: the cross-signed root shows as an **"extra download"** (the server isn't sending it), and the report isn't **A+** until the chain is complete. Their **other subdomains** (on Cloudflare, not Launch) presented **one complete chain** and validated fine — confirming it's a **chain-completeness config issue, not a bad cert**.

## Fix applied — add the cross-signed cert to the chain (dev + prod)

- Launch **added "Sectigo Public Server Authentication Root R46 x USERTrust RSA CA" to the certificate chain for the dev and prod domains** and asked the customer to re-verify.
- ⚠️ **First attempt did NOT fully resolve it** — old browsers still showed the SSL error after adding the cross-chain, so a **call with the customer** was set up to work through the remaining steps (likely chain ORDER / which intermediates are bundled / propagation). **Treat "add the cross-signed root" as necessary but verify the full presented chain end-to-end** (order + completeness) on an actual old client and via SSL Labs.

## Also requested — enable HSTS

- The customer also asked Launch to **enable / help enable HSTS** as part of getting to an **A+ SSL Labs** score.

## Generalized guidance (for future similar queries)

- **"Site not secure / untrusted on OLD browsers only, fine on new ones"** = **incomplete cert chain**: the server isn't presenting an intermediate/**cross-signed root** that old trust stores need. **Newer browsers have the newer root natively; old ones don't** → they need the **cross-signed** path to an older, widely-trusted root.
- **Fix = add the cross-signed root to the presented chain** (e.g. Sectigo R46 **cross-signed with USERTrust RSA CA**). Sources like **Sectigo's intermediate-certificates page** provide the cross-signed bundle. Get **SSL Labs to A+** (no "extra download," complete chain) and **verify on a real old device** — SSL Labs alone may not fully mirror an old client.
- **Adding the cross-signed cert may not be a one-shot fix** — chain **order** and **which intermediates are bundled** matter; be ready to iterate on a **dev/lower environment** and confirm on the actual failing browser.
- **Don't "solve" it by telling a bank to allow older browsers** — for a financial institution that's a **security risk**; the correct fix is a **complete, cross-signed chain** that old clients validate.
- Related: [.bank TLD double-redirect → single redirect (same customer, FIB)](launch-bank-tld-double-redirect-violation-single-redirect-apex-to-https-www.md), [apex cert renewal / move apex to Launch (FIB)](launch-apex-cert-renewal-expired-redirect-move-apex-to-launch.md), [add SAN domains / managed cert / DCV](launch-add-san-domains-managed-cert-100-limit-format-renewal-cname-dcv-7day.md), [cutover cert + hostname validation](launch-cutover-lets-encrypt-cert-hostname-validation-txt-apex-vs-www-jax.md).

## Status

- **In progress → escalated (major issue, financial institution).** Cause = the presented **cert chain lacked the cross-signed root**, so **old browsers (old-Mac Safari/Firefox) couldn't validate** and showed "not secure." Launch **added "Sectigo Public Server Authentication Root R46 x USERTrust RSA CA" to the chain on dev + prod**, but **old browsers still errored** on first verify → **customer call scheduled (Monday MT)** to finish the chain config (order/bundle) and **enable HSTS**, targeting an **SSL Labs A+**. Customer had the **same issue on their previous host** (config, not a Launch-specific bug).
