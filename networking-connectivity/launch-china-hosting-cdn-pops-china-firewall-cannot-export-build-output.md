QUESTION

An **SI partner** wants to host a **Next.js ISR frontend on Launch** with a **local presence in Mainland China** (to serve the Chinese audience and avoid the **Great Firewall** blocking, per Contentstack's "delivering content within China" guidance). Two questions:
1. **Does the Launch CDN have Points of Presence (POPs) inside China?**
2. **Can they get the generated build output (or partial output) OUT of Launch** to transfer/host those files at a location within China? (Automate only exposes **Deploy a Build** and **Revalidate CDN cache**.)

_Keywords: host Launch in China Mainland, CDN POPs in China Cloudflare, Great Firewall blocking, deliver content within China separate frontend instance, extract build output files from Launch, .next directory export, Automate Deploy a Build Revalidate CDN cache only, no build output download, expected outbound bandwidth from China, POPs provisioning China._

ANSWER

## 1. POPs in China — being worked on with Cloudflare (not available by default)

- **Where content is served from is determined by the CDN's POPs.** Launch's CDN is **Cloudflare**; **China POPs are not enabled by default.**
- Launch is **coordinating with Cloudflare to enable provisioning for access to POPs in China.** To scope/prioritize this, the customer should share **their expected outbound-traffic bandwidth from China.**
- Cloudflare **does have POPs in China** (subject to their China-network requirements/ICP licensing), so enabling this would let Launch serve the Chinese audience directly — treat as **in-progress / provisioning-dependent**, not a current guarantee.

## 2. Getting build output OUT of Launch — not exposed

- **Launch does not provide a way to download/export the generated build output** (e.g. the `.next` directory). The **Automate Launch actions are limited to `Deploy a Build` and `Revalidate CDN cache`** — there's **no "fetch/export output files" action**, and no other supported mechanism to pull the build artifacts out for hosting elsewhere.
- So the "**build on Launch, then copy files into a China-hosted instance**" pattern **isn't supported** via Launch. The China-presence options are:
  - **Enable China POPs on the Launch CDN** (the path being pursued with Cloudflare), or
  - Follow Contentstack's documented **"delivering content within China"** approach — typically a **separate frontend instance deployed inside China** (built from the same source, fetching content via the Delivery/CDA layer), rather than extracting Launch's build output.

## Generalized guidance (for future similar queries)

- **"Can Launch serve inside Mainland China?"** → depends on **Cloudflare POPs in China**, which are **not on by default**; Launch is **working with Cloudflare to provision China access.** Ask for **expected outbound bandwidth from China** to scope it. Not a current guarantee.
- **"Can we export the build output (`.next`) from Launch to host it elsewhere?"** → **No** — Launch exposes only **Deploy a Build** and **Revalidate CDN cache** (via Automate); there's **no build-artifact export.** For a China presence, either **enable China POPs** or run a **separate in-China frontend instance** per the "delivering content within China" docs (build from source there; don't try to lift Launch's output).
- Related: [global scaling / regions / multi-region limit (edge + CDN)](../deployments-builds/launch-global-scaling-regions-available-clouds-multi-region-limit-edge-cdn-rfp.md), [cross-environment failover / multi-region HA architecture](../edge-functions-rewrites/launch-cross-environment-failover-edge-function-multi-region-ha-architecture.md), [ISR support + revalidation cap](../caching-cdn/launch-isr-support-cache-revalidation-limit-cap.md), [no fixed outbound/egress IP](launch-no-fixed-outbound-egress-ip-allowlist.md).

## Status

- **In progress / partial answer.** **China POPs:** **not enabled by default**; Launch is **coordinating with Cloudflare to provision China POP access** — requested the customer's **expected outbound bandwidth from China** to scope. **Build-output export:** **not supported** — Automate offers only **Deploy a Build** + **Revalidate CDN cache**, no way to pull the generated files out. For a China presence, **enable China POPs** or use a **separate in-China frontend instance** per the "delivering content within China" docs. (Partner noted Cloudflare's China POPs would make Launch compelling for SEA customers.)
