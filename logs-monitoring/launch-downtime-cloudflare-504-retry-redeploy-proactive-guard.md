QUESTION

Customer **UVA Health System** (SF case **00056929**, Org `bltdcc012133cba1c83`) reported their site **went down again** with the **same message seen previously** (Azure Container App Unavailable) that they'd been told was fixed. Timing detail: a **failed deploy (#4925)** occurred **~28 minutes** earlier, but the **site became unavailable only ~8 minutes ago** — so the outage didn't line up with the failed deploy.

_Keywords: site down again, Azure Container App Unavailable, Cloudflare 504, downtime, failed deploy timing, retry network issues, redeploy restored, proactive guard, recurring outage._

ANSWER

## Cause — Cloudflare 504 errors (with retry network issues)

- The downtime was caused by **Cloudflare errors with 504 status codes**. Additionally, **network issues were observed when Launch attempted retries**, compounding it.
- The **failed deploy (#4925) ~28 min earlier did not directly cause it** — the **site went down ~8 min ago**, correlating with the **Cloudflare 504s**, not the deploy.
- This is a **Cloudflare-layer / upstream** issue (Cloud team investigating), not the customer's code.

## Resolution

- The customer **redeployed**, and the **site came back up.**
- Launch is **investigating how to avoid downtime even when Cloudflare fails**, and working with the **Cloud team** on the **retry network issues**.
- A **proactive (optional) guard from the Launch side** is **planned** to mitigate this class of failure; it was added to the **stability** work items. Customer is **unblocked** in the meantime.

## Generalized guidance (for future similar queries)

- **"Site down again with 'Azure Container App Unavailable' / same recurring message"** → check for **Cloudflare 504s** at the edge; a **redeploy** typically restores it. The cause is **upstream (Cloudflare/edge)**, not necessarily the app.
- **Don't over-attribute to a nearby failed deploy** — correlate **exact timestamps**: if the site went down well after the failed deploy, the deploy likely isn't the cause (here, CF 504s were).
- **Mitigation** — a redeploy restores service; Launch is adding a **proactive guard** for Cloudflare-failure resilience. Related UVA outages: [CF001 resource saturation](launch-cf001-server-failed-to-start-resource-saturation-memory-leak.md), [Azure regional incident](launch-downtime-no-server-logs-azure-regional-incident.md).

## Status

- Cause: **Cloudflare 504s** (+ retry network issues); customer **redeployed → restored**. Cloud team investigating; Launch **planning a proactive optional guard** (added to stability points). Customer unblocked.
