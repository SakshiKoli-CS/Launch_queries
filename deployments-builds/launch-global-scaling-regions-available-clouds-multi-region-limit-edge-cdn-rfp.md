QUESTION

RFP question: **How does Launch support global scaling across multiple regions/countries?** Any **limitations for high-traffic, multi-region deployments**? Follow-up: **can we start in a few regions and easily expand?**

_Keywords: global scaling multi-region, available cloud providers regions AWS Azure GCP NA EU AU, multi-region deployment not supported simultaneously, zone redundancy high availability, autoscaling high traffic no downtime, Edge Functions POPs global, edge caching static pages, RFP scalability, CDN global delivery._

ANSWER

## Choose your cloud + region

- Launch lets you choose the **cloud provider and region** for hosting. Available options: **AWS NA, AWS EU, AWS AU, Azure NA, Azure EU, GCP NA, GCP EU.**

## High availability + scaling

- Applications are **redundant across zones within a region** → **high availability.**
- **Scaling is fully managed by Launch** and **scales very quickly for high traffic with no downtime.**

## Limitation — no simultaneous multi-region for one application

- **Launch does NOT support deploying the same application across multiple regions simultaneously.** HA is **in-region multi-AZ**, not active-active across regions.

## Global performance — Edge + CDN

- **Edge Functions** run custom logic **closer to users at the edge**, distributed across **hundreds of global points of presence (POPs)** → better performance / lower latency.
- **Static pages cache at the edge** (CDN), further improving load times globally — so even though the app runs in one region, **delivery is global via the CDN/edge.**

## On "start small and expand"

- You **pick a region per environment/app** and Launch **autoscales within it**; you can **add environments/apps in other available regions** as you grow. There is **no single-app active-active multi-region**, so "global presence" comes from **edge/CDN delivery** (and, if true regional redundancy is required, a DIY multi-environment setup — see cross-link).

## Generalized guidance (for future similar queries)

- **RFP "global/multi-region scaling?"** → Launch = **pick one of AWS/Azure/GCP × NA/EU/AU(AWS only)**, **multi-AZ HA in-region**, **fully-managed fast autoscaling, no downtime**. **No simultaneous multi-region for a single app.** **Global reach = Edge Functions across hundreds of POPs + edge/CDN caching of static content.**
- **"Start small, expand later"** → yes, choose a region now and add environments/regions later; true cross-region redundancy is **customer-architected** (multiple environments + failover), not built-in.
- Related: [cross-environment failover / multi-region HA architecture](../edge-functions-rewrites/launch-cross-environment-failover-edge-function-multi-region-ha-architecture.md), [scalability/compute & SLA reference](launch-scalability-compute-limits-large-ssg-builds-runtime-throughput-sla.md), [no static IPs / regions](../networking-connectivity/launch-no-static-ip-no-vpn-secure-db-api-via-auth-cloud-functions-azure.md).

## Status

- **Answered (RFP).** Regions: **AWS NA/EU/AU, Azure NA/EU, GCP NA/EU**; **multi-AZ HA in-region**, **fully-managed autoscaling (no downtime)**; **no simultaneous multi-region for one app**; **global performance via Edge Functions (hundreds of POPs) + edge/CDN caching.** Can **start in a region and expand** with more environments/regions over time.
