QUESTION

Customer (Lee, for an **RFP**) asks: does Launch **support/recommend an Edge Function to provide availability/failover across two Launch environments** (sample AI-generated edge function: try PRIMARY origin with a timeout → on failure, sticky-failover to SECONDARY)? Alternative is an **external load balancer** in front of Launch. Also: is **multi-region deployment** coming (AI claimed it's "in deployment")?

_Keywords: edge function failover two environments, cross-region cross-cloud failover, primary secondary origin sticky failover, multi-region multi-cloud deployment Launch, high availability multi availability zone single region, external load balancer in front of Launch, RFP availability, AWS NA Azure NA failover._

ANSWER

## Current Launch HA architecture

- **Launch deploys across multiple availability zones within the same region** → **high availability at the regional level.**
- **Launch does NOT offer multi-region or multi-cloud deployments for a single environment**, and that's **not on the immediate roadmap** (so the AI claim that multi-region is "in deployment" is **not accurate**).

## On the cross-environment failover edge function

- The sample edge function routes between **two completely separate Launch environments** (e.g. one on **AWS NA**, one on **Azure NA**) — i.e. a **cross-cloud / cross-region failover** the customer would build themselves.
- This is **not a built-in Launch feature.** If a customer genuinely needs whole-region/cloud failover beyond Launch's in-region multi-AZ HA, it would require **them** to:
  1. **Subscribe to Launch hosting on two different cloud providers/regions.**
  2. **Keep both environments synchronized.**
  3. **Implement their own failover logic at the edge** (like the sample) **or use an external load balancer** in front of Launch.
- Clarify the intent first: **cross-region/cross-cloud failover** (region outage → fail to another) vs **latency/performance across geographies** — they need different designs.

## For most cases, in-region multi-AZ HA is sufficient

- Built-in **multi-AZ-within-region** HA is often enough (the customer here agreed it suffices — a global/UK-specific delivery option would be overkill). Reach for the DIY two-environment + edge/LB failover only when **cross-region/cross-cloud resilience** is an explicit requirement.

## Generalized guidance (for future similar queries)

- **"Does Launch do multi-region / cross-cloud failover for one environment?"** → **No** (and **not on the near-term roadmap**). Launch provides **multi-AZ HA within a single region**.
- **Cross-region/cross-cloud failover is DIY:** two separately-subscribed Launch environments (different cloud/region) + **synchronization** + your **own edge-function failover or an external load balancer**. Launch doesn't manage it.
- **Set RFP expectations accordingly:** regional HA = built-in; geo-redundancy / active-active across regions = customer-architected.
- Pattern for a self-built failover edge function: **try primary with a timeout → on error/non-OK, sticky-failover to secondary** (TTL-based stickiness via the edge cache), as in the customer's sample.
- Related: [simulate outage to test failover (edge function)](launch-simulate-outage-edge-function-error-failover-status-codes.md), [Cloudflare 504 downtime / proactive guard](../logs-monitoring/launch-downtime-cloudflare-504-retry-redeploy-proactive-guard.md), [scalability/SLA reference](../deployments-builds/launch-scalability-compute-limits-large-ssg-builds-runtime-throughput-sla.md).

## Status

- **Answered.** Launch = **multi-AZ HA within one region**; **no multi-region/multi-cloud for a single environment** (not on near-term roadmap; AI "in deployment" claim incorrect). **Cross-environment failover (the edge-function sample) is DIY** — needs two subscriptions on different clouds/regions + sync + customer-built edge/LB failover. Customer concluded **in-region multi-AZ is sufficient** for their (UK) use case.
