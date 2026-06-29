QUESTION

Customer **Rapid7** (Org `blte4f029e766e6b253`, SF case **00052341**) runs Launch behind **AWS CloudFront** (CloudFront proxies to Launch). They want to **verify their failover** for when Launch is unavailable (e.g. the Nov 18th Cloudflare outage). They ask to:

1. **Simulate a Launch outage in their development environment.**
2. Know **which response statuses to cover** for failover (they currently trigger on **500, 502, 503, 504**).

_Keywords: simulate Launch outage, test failover, CloudFront in front of Launch proxy, Cloudflare down, edge function return 500, failover status codes 500 502 503 504, dev environment outage simulation, validate failover logic._

ANSWER

## Simulate an outage with an Edge Function that returns an error

- Rather than waiting for a real outage, deploy an **Edge Function in the development environment that intentionally returns an error response** for all requests — this **mimics Launch being unavailable** and lets them validate CloudFront's failover behavior. Example returning a 500:
  ```js
  export default async function handler(request, context) {
    return new Response("Internal Server Error", { status: 500 });
  }
  ```
- Change the `status` to exercise each failover code (502/503/504) as needed.
- This can be handled **async** — a call isn't required up front; offer one only if questions remain.

## Status codes to cover for failover

- The customer's existing set — **500, 502, 503, 504** — is the relevant range to mimic/cover for a Launch-unavailable scenario. Validate failover against each.

## Generalized guidance (for future similar queries)

- **"How do we simulate a Launch outage / test our CDN failover (CloudFront/Akamai in front of Launch)?"** → deploy an **Edge Function in a non-prod environment that returns an error status** (`500`/`502`/`503`/`504`) for all requests; point the failover test at that env. No real outage needed.
- **Failover trigger codes:** the **5xx family relevant to upstream unavailability — `500, 502, 503, 504`** — is the set to cover.
- An Edge Function returning a fixed `Response(..., { status })` is the simplest, controllable way to inject any HTTP status for testing edge/CDN behavior.
- Related: [block default domain / edge example patterns](launch-custom-404-static-nuxt-ssg-edge-function.md), [Cloudflare 504 downtime (proactive guard)](../logs-monitoring/launch-downtime-cloudflare-504-retry-redeploy-proactive-guard.md).

## Status

- **Answered (async).** To **simulate an outage**, deploy a dev-environment **Edge Function returning an error status** (e.g. 500; vary for 502/503/504) and validate CloudFront failover against it. Confirmed the **500/502/504/503** set is appropriate for failover coverage. Call deferred unless follow-up questions arise.
