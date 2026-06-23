QUESTION

Customer **UVA Health** (account: WillowTree Inc) — sites `uvahealth.com` and `childrens.uvahealth.com`, both on **Launch** — went **down for ~10 minutes starting 8:16 AM ET on May 29, 2026**. Their Launch logs showed **"No server logs to display"** during that window. Was there an **infrastructure event, restart, or incident on Launch** around that time? Any internal logs / post-mortem to reference?

Follow-up from the customer: the Contentstack status incident they found **began hours earlier and showed resolved at 06:18 MDT**, which seemed to *precede* their outage — so they questioned whether it was related, and whether the **resolution process itself** triggered their issue.

_Keywords: Launch downtime, site outage, no server logs to display, 504 Gateway Timeout, Azure North America incident, US West 2 regional disruption, upstream cloud provider, status page incident, post-mortem, intermittent unavailability after deploy._

ANSWER

## Cause — upstream Azure regional incident (not a Launch-specific fault)

- The downtime aligns with an **Azure North America (US West 2) regional disruption** that began **~05:20 UTC on May 29, 2026**, impacting **Virtual Machines and related service operations**. Customers on Azure NA saw **intermittent failures and brief unavailability**.
- Reviewing Launch **observability logs for the reported window** showed **504 Gateway Timeout** errors — consistent with **transient upstream infrastructure impact**, including brief downtime following deployments.
- This is an **upstream cloud-provider issue**, not a Launch platform defect. Reference: **Contentstack Status** page — incident: `https://status.contentstack.com/incidents/57l45wjs31rz` (Azure NA – Proactive Monitoring of Upstream Cloud Provider Issue).

## Resolving the timeline confusion (status "resolved at 06:18 MDT" vs the 8:16 AM ET outage)

- The Contentstack **status update** was posted/resolved at **06:18 MDT**, but **Microsoft's incident report shows recovery activities in Azure West US 2 continued until 02:30 UTC on May 30 (08:30 PM MDT, May 29).**
- So the customer's outage **falls within the broader Azure incident and recovery window**, even though the Contentstack status entry was marked resolved earlier. Microsoft noted **intermittent connectivity, elevated latency, and service-accessibility problems** across the region during recovery.
- Reference: **Microsoft Azure Status History – West US 2** incident details.

## Why "No server logs to display" during the window

- When the impact is **upstream (504 Gateway Timeout — request never completes at the origin)**, the application may produce **no server logs** for that period; the failure is at the gateway/infrastructure layer, not in app execution. Combined with Launch's **short log retention**, a past window can show empty.

## Generalized guidance (for future similar queries)

- **"Our Launch site went down for N minutes — was there a platform incident?"** → Check the **Contentstack Status page** first, and correlate with the **upstream cloud provider's** status history (Azure/AWS). Many brief outages trace to **regional cloud-provider incidents**, not Launch itself.
- **504 Gateway Timeout + "No server logs to display"** → signature of **upstream/infra impact** (request didn't reach/complete at origin), not an application error.
- **Status "resolved" time can predate a customer's outage** — the provider's **full recovery window** often extends past the status-page resolution timestamp; compare against the **cloud provider's own incident report** (and watch timezone conversions: MDT vs UTC vs ET).
- Launch **log retention is short** — capture logs during a live incident; retrospective server logs may be unavailable.

## Status

- Explained to the customer with status-page and Microsoft incident references; attributed to the **Azure US West 2 regional incident + extended recovery window**. Azure incident since **resolved**, infrastructure back to normal with no further impact.
