QUESTION

Customer **UVA Health** (Org `bltdcc012133cba1c83`, Azure North America, SF case **00053918**) reports **slowness across their whole site** (`uvahealth.com`) plus **errors on some pages**. They ask whether there's a **larger Azure issue** CS is aware of, and request **logs / cluster health**.

Details:
- **Slow load times across all pages** on the main site.
- **Errors on dynamically-built provider profile pages** (built via API connection), e.g. `https://www.uvahealth.com/providers/Lawrence-Borish-1316016934`:
  ```json
  {"message":"Internal Server Error CF001. Server failed to start. Please check Launch Server Logs for more details."}
  ```
- Customer **confirmed they see no errors in Launch Server Logs.**

_Keywords: site slow all pages, CF001 Server failed to start, provider profile pages dynamic API, CPU memory 100% spike, resource saturation, redeploys masked the issue, container app not found reduced deploys, increase CPU memory containers, no caching enabled, enable caching performance._

ANSWER

## Cause — application-level resource saturation (CPU/memory → 100%)

- This is **application-specific behavior**: the app's **CPU and memory utilization spike to 100%**, which causes the **slowness** and the **CF001 ("Server failed to start")** on the dynamic provider pages.
- **Why it surfaced now:** previously, **frequent/periodic redeployments effectively reset the containers**, masking the buildup. After **deployments were reduced** (due to the **"container app not found"** issue), the **underlying resource consumption became visible**.
- This is the **standard resource allocation** that works across clouds/regions for Launch customers — i.e. not an Azure-wide outage.

## Mitigation applied

- **Increased CPU and memory** for the UVA Health application containers → **resource usage no longer spiking.** Should address the issue going forward.

## Strong recommendation — enable caching

- The site has **no caching enabled.** **Implementing caching would significantly reduce load and improve performance** (fewer dynamic renders hitting the resource-constrained containers).

## Generalized guidance (for future similar queries)

- **"Site-wide slowness + CF001 on dynamic pages, but no errors in Server Logs, asking about an Azure outage"** → most likely **app-level CPU/memory saturation (100%)**, **not** a platform/Azure outage. CF001 here is a **symptom of the container failing to start under resource pressure**, not a separate bug.
- **Watch for a masking pattern:** **frequent redeploys silently reset containers** and hide a memory/CPU buildup; when deploy frequency drops, the latent saturation **suddenly surfaces**. Don't mistake "it just started" for a new platform regression.
- **Mitigations:** (1) **raise container CPU/memory**; (2) **enable caching** to cut dynamic load — dynamic, API-built pages (like provider profiles) are prime caching candidates.
- Related UVA incidents (separate threads/causes): [CF001 resource saturation / memory leak](launch-cf001-server-failed-to-start-resource-saturation-memory-leak.md), [Cloudflare 504 downtime](launch-downtime-cloudflare-504-retry-redeploy-proactive-guard.md), [Azure regional incident](launch-downtime-no-server-logs-azure-regional-incident.md).

## Status

- Cause: **app-level CPU/memory saturation** (surfaced after redeploys dropped due to the "container app not found" issue) — **not** an Azure outage. **Mitigated by increasing container CPU/memory** (spikes stopped). **Recommended enabling caching** for durable performance improvement.
