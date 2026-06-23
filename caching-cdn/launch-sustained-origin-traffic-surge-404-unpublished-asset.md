QUESTION

Customer **West Monroe Partners** (Support Case **#00058977**) reported a large, sustained surge in **CMS API usage and Launch origin traffic**, investigated across several related questions:

1. **Broken external image / unpublished entry, served at high volume:** A 404ing external image asset and an unpublished article URL were being requested at very high volume (~328,743 requests / 30 days) **despite the source entry being unpublished since October 2025** and the URL no longer appearing in the entry content. Customer asked: is Launch caching a stale reference or build artifact?
   - Asset (404): `https://www.wmcapital.com/wp-content/uploads/2024/10/wm_h_pos_clr_rgb_August2024_158x33.jpg`
   - Source entry: "How to migrate attachments from..." at `/insights/how-to-migrate-attachments-from-...` — *Not Published*
2. **CDN expansion theory:** API consumption **quadrupled around May 14, 2026** with no code changes. Customer hypothesized Launch added edge PoPs / changed origin-fetch infra, so that with their `Cache-Control: s-maxage=86400, stale-while-revalidate=60`, more edge nodes independently revalidate → 4x origin requests.
3. **Unattributable 24/7 surge:** A 4.5x API volume increase, **constant ~50,000–65,000 req/hr, 24/7 including overnight** (no diurnal/human shape), while their **Next.js cache misses actually decreased** (23,420 → 13,320). Customer ruled out code changes and external user traffic and asked for platform-level guidance.

- **Stack API key:** `blt88fb2beacdfabc05`
- **Org ID:** `blt4ac2fdd12da7134a` (AWS NA)
- **Project:** WestMonroeWebsite · **Environment:** production

_Keywords: sustained API surge, origin traffic spike, 404 high volume, unpublished entry still requested, stale cache reference, build artifact, CDN tiered cache, edge PoP revalidation, s-maxage stale-while-revalidate, node-fetch traffic, Microsoft Office user agent, link preview crawlers, bandwidth spike, traffic logs._

ANSWER

## 1. 404 traffic on unpublished entry / image asset — NOT a Launch caching issue

Investigated requests for both the unpublished article URL and the image asset URL:

- **Unpublished article** (`/insights/how-to-migrate-attachments-from-...`): although unpublished in Oct 2025, requests continue from **multiple external IPs and user agents**, returning **404**. This is traffic to a **historical URL still known to search engines, crawlers, caches, and third-party systems** — not Launch re-serving it.
- **Image asset** (`/wp-content/uploads/.../...jpg`): traffic originates from **Outlook/Office clients, Apple Mail clients, and crawler/preview services**. **Cloudflare analytics show no page views, visits, or API requests** for the image path — i.e., the URL is requested **directly**, not via active page rendering or CMS activity. Also returns 404.

**Finding:** No Launch-side caching behavior, stale content artifact, or platform issue. The continued traffic is **external systems still holding references to historical URLs.** An unpublished entry does not stop external clients from requesting a URL they already know — those requests just 404.

## 2. CDN expansion / edge revalidation theory — does NOT explain the spike

- Launch uses **CDN Tiered Cache**. With Tiered Cache, **lower-tier edge PoPs do NOT independently revalidate against origin** when objects expire — they fetch from designated **upper-tier caches**, and only the upper-tier talks to origin when necessary.
- Therefore **adding edge PoPs does not produce a proportional increase in origin revalidation** — Tiered Cache specifically consolidates cache misses / revalidation through the upper tier to *reduce* origin fetches.
- **No known Launch infrastructure change around May 14, 2026** that would make every edge location revalidate independently. The customer's `s-maxage`/`stale-while-revalidate` × "more edge nodes" theory does not hold under a tiered-cache architecture.

## 3. Sustained, programmatic 24/7 surge — platform guidance

The contradiction the customer raised is real and points **away from their app**: their **Next.js cache misses decreased** (fewer origin fetches from the app) while **total API volume rose 4.5x**, and the **traffic is flat 24/7 with no diurnal shape** — inconsistent with user-driven traffic.

Findings / recommendations from Launch:

- **No known Launch platform behavior or internal service** generates sustained, programmatic `node`/`node-fetch` traffic against a hosted stack.
- **Platform telemetry observed**: requests from **Microsoft Office-related user agents**, suggesting some traffic is from **external tooling, automation, link-preview services, or client-side integrations** rather than Launch. The increase appears to have started **around May 11** (slightly before the customer's noted May 14).
- **Enhanced visibility is limited today**: Launch's **Traffic Logs** feature (on the roadmap) will give better request/traffic-pattern visibility for investigations like this. A platform-level traffic report was shared in the interim.
- **Note on log retention**: Launch application logs are retained **only 24 hours**, so retrospective analysis of a past spike date is not possible after the fact — capture data while the event is live.

**Recommended path forward:**
1. Temporarily **restrict/block the specific endpoint with an Edge Function** and monitor the resulting traffic + API consumption — this isolates whether the traffic is external. (Ref: Edge Functions docs.)
2. **Audit any automations, integrations, monitoring tools, or internal systems** that might call the endpoint; stopping/adjusting them may reduce the traffic.

## Generalized guidance (for future similar queries)

- **"Why is Launch still serving/requesting an unpublished entry or deleted asset?"** → Launch isn't. External crawlers/mail clients/preview services keep hitting **historical URLs they already know**; those requests **404** but still count as traffic. Unpublishing does not retract external references.
- **"Did adding CDN edge nodes multiply our origin requests?"** → Under **Tiered Cache**, no — lower tiers revalidate via upper-tier caches, so edge expansion does not proportionally increase origin fetches.
- **"Sustained, flat, 24/7, non-diurnal API traffic with no code change"** → Strong signal of **automated/external clients** (Office/mail/preview bots, monitoring, integrations), not user traffic and not a Launch internal process. There are no known Launch services that generate `node-fetch` traffic against a stack.
- **Investigation tooling**: Launch app logs last **24h only**; **Traffic Logs** is roadmap; use an **Edge Function to block/observe** a suspect endpoint as the practical next step.
- **Cache-miss vs total-volume contradiction** (misses down, volume up) → indicates the calls are **not** the app's origin fetches responding to users; look to external direct requesters.

## Status

- Findings communicated to the customer at each stage. Investigation ongoing on attributing the exact external source; recommended next step is the Edge Function block-and-observe plus an audit of integrations/automations. Awaiting further customer data.

### Update — meeting + Edge Function block not yet implemented

- **Customer requested a meeting** to further discuss the API/bandwidth surge and review findings (customer team in **EDT**, IST − 9.5h); a slot was offered for scheduling.
- **Key follow-up finding:** a **significant volume of requests is still hitting origin** for the image asset `/wp-content/uploads/2024/10/wm_h_pos_clr_rgb_August2024_158x33.jpg`. The earlier recommendation to **block this URL via an Edge Function does not appear to have been implemented yet** — confirmation was requested from the account team.
- **Takeaway:** the recommended Edge-Function block/observe step is the gating action — until it is actually deployed, the origin traffic for the flagged asset continues, and the external-source attribution can't be conclusively validated. Confirm implementation before treating the recommendation as exhausted.
