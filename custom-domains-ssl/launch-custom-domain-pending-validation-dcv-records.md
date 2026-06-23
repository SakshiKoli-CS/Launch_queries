QUESTION

Customer **Ericsson / Cradlepoint** reported two custom domains stuck in **"Pending Validation"** status on their `prod-ews` Launch environment:

- `cradlepoint.ericsson.com` — DNS changes made a few months ago
- `resources.cradlepoint.dev` — DNS changes made May 27; CNAME points to `cp-prod.contentstackapps.com`

Their DNS team was looking for the **DCV (Domain Control Validation) Name/Value** required for certificate validation. They asked Launch to verify: (1) whether pending certificate-validation requests exist for these domains, (2) whether specific **DCV records (Name/Value)** were generated and need to be shared, and (3) why the domains remain in Pending Validation.

_Keywords: custom domain pending validation, Domain Control Validation, DCV record, TXT record, certificate validation stuck, hostname status pending, CNAME contentstackapps.com, SSL certificate not issued, Cloudflare validation delay._

ANSWER

## What was found

- **`resources.cradlepoint.dev`** — **certificate and hostname validation both succeeded.** It cleared to a validated state without any action needed on the Launch side; the **delay appeared to originate from the Cloudflare end** (Launch did nothing special to fix it). Customer confirmed this one resolved.
- **`cradlepoint.ericsson.com`** — **still Pending Validation.** Next step recommended: ask the customer to **verify the TXT and DCV records for this domain were added correctly** at their DNS provider. (Customer did not reply to follow-ups; left open.)

## Why a domain stays in "Pending Validation"

- The certificate cannot be issued until the **DCV records (and any TXT/validation records) are correctly added at the customer's DNS provider** and have propagated.
- A common cause of a *stuck* domain (vs. a wrong-config one) is **propagation / upstream (Cloudflare) validation delay** — sometimes it clears on its own once DNS/cert validation completes upstream. Don't assume a config error before checking record correctness + propagation.

## Important note on RCA

- For the `resources.cradlepoint.dev` case, the delay was traced to the **Cloudflare side**, and Launch did not make any change to resolve it. **Do not commit to a specific RCA with the customer** until confirmed with Cloudflare.

## Resolution

- `resources.cradlepoint.dev`: **resolved** (validation succeeded; delay was upstream/Cloudflare).
- `cradlepoint.ericsson.com`: **open** — pending the customer verifying their TXT/DCV records; no customer reply at last update.

## Generalized guidance (for future similar queries)

- **"My custom domain is stuck in Pending Validation / no certificate"** → First **verify the DCV + TXT validation records are correctly added** at the customer's DNS provider and have propagated. Certificate issuance is gated on these.
- **DCV Name/Value** records are required for certificate validation; if the customer's DNS team is "looking for the DCV record," confirm what Launch generated for that domain and that it matches what's in DNS.
- **Stuck-but-correct domains** can clear on their own due to **upstream (Cloudflare) validation/propagation delay** — re-check status before deeper investigation, and **don't commit to an RCA** until the upstream cause is confirmed.
- Treat each domain independently — one domain validating successfully does **not** mean a sibling domain's records are correct.
