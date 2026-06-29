QUESTION

A **prospect running a POC** wants to do **performance/load testing** on Launch. Is it allowed? How long can they run a test, and what type? What should be asked to set **scope, boundaries, and timelines**? (Also: the demo app being tested is **not production-grade** — should advise using their own frontend.)

_Keywords: load testing allowed, performance testing prospect POC, must be pre-approved coordinated, Load Testing Guide checklist, monitor platform stability, scope boundaries timelines, demo app not production grade, inform Launch before load test._

ANSWER

## Yes — load/performance testing is allowed, but it MUST be pre-approved & coordinated

- **Load and performance testing is allowed**, but it **must be coordinated with the Launch team beforehand and pre-approved** so the team can **monitor and ensure platform stability** during the test.
- The process: **complete the checklist** in the official **Contentstack Load Testing Guide**, provide the requested details, and the Launch team **reviews and approves** before the test runs.
- Docs: **Contentstack Load Testing Guide** (`/docs/developers/launch/load-testing`).

## Setting scope / boundaries / timelines

- Don't free-form the parameters — the **Load Testing Guide checklist** is what defines the **scope, load profile, timing, and target** to share for approval. Have the prospect fill it out (expected RPS / concurrent users, ramp-up, duration, target hostname/endpoints, test windows, region).

## Demo app caveat

- If the test will hit the **Contentstack demo app**, note it's **not production-grade** — results won't represent production performance. **Advise the prospect to use their own frontend app** where possible for a meaningful test.

## Generalized guidance (for future similar queries)

- **"Can we run a load/performance test on Launch?"** → **Yes, but only pre-approved + coordinated** so Launch can monitor. **Never run unannounced** (unannounced bursts look like incidents/DDoS and trip rate limits — see related load-test cases).
- **Use the Load Testing Guide checklist** to define scope/load/timeline/target and get sign-off; that's the authoritative scoping tool.
- **Test against the customer's own production-grade frontend**, not the demo app, for representative numbers.
- Related: [load-test 429 / uncached origin (coordinate first)](../caching-cdn/launch-loadtest-429-uncached-origin-cache-control-headers-moncler.md), [intermittent TCP ETIMEDOUT under load / autoscaling](../networking-connectivity/launch-intermittent-tcp-etimedout-econnrefused-cda-autoscaling-spike.md), [load-test response-time spike](launch-load-test-response-time-spike-buffered-responses.md).

## Status

- **Answered.** Load/performance testing is **allowed but must be pre-approved and coordinated** (Launch monitors for stability). Direct the prospect to **complete the Load Testing Guide checklist** for review/approval; **recommend their own (production-grade) frontend** over the demo app.
