QUESTION

Customer **Philips / EPAM** (SF case **00045331**, JIRA **CL-2033**, Project `67a5d6832181bda09f767cb4`, EU, Next.js + `next-intl` i18n) hits **CF001 "Internal Server Error CF001. Server failed to start"** / **HTTP 500** intermittently on UAT. Symptoms: the server sometimes fails and **tries to restart, but can't bind because the port (4000) is already in use** ("Address already in use"). They report **unhealthy instances still receiving traffic** (~0.04% 500/502 under load; one user 500s while others succeed on the same page). They ask, in depth:
1. **How does Launch determine an app is healthy?** What checks, and at what stage (deploy / before routing traffic to a new node / etc.)?
2. **What's the recovery strategy** when an app is unhealthy?
3. Are there **Lambda resource limits** (CPU/memory/concurrency) to know about?
And they push hard on **readiness vs liveness** being platform responsibilities.

_Keywords: CF001 server failed to start, address already in use port 4000, how Launch determines application health, health check port 4000 Pingdom, readiness liveness node readiness platform, Launch runs on AWS Lambda no long-lived nodes, no health check before routing traffic, restart loop unhandled error crash, unhealthy instance receiving traffic, x-forwarded-host does not match origin Server Actions aborting, MODULE_NOT_FOUND crash, fails on contentstackapps.com works on real domain Akamai headers, block default domain, /health endpoint CI/CD safeguard, x-deployment-uid x-environment-uid._

ANSWER

## How Launch determines application health

Launch checks health in **two ways**:
1. **Port check** — verifies the app is **listening on port 4000**. If it isn't, Launch **restarts the process**.
2. **Pingdom checks** — real-time monitoring of **production** apps; **non-success responses trigger alerts** (usually pointing at application-code issues).

## Recovery model + why the restart loop / "port 4000 in use"

- **Recovery = restart.** If Launch deems the app unhealthy (not on port 4000), it **restarts** it.
- **The restart loop:** the app may **start but fail to respond correctly due to its own logic** (unhandled error / crash), so Launch **tries multiple restarts** → the "Address already in use / failed to start (port 4000)" logs. **Normally harmless unless the app consistently fails to start.** The fix is on the **app side**: **fail fast on startup** if a required dependency (DB/API) is unavailable rather than running broken, and **handle errors** so an unhandled throw doesn't crash the process.

## Readiness / liveness — Launch runs on AWS Lambda, so the container/VM mental model doesn't apply

The customer argued Launch lacks **node-level readiness/liveness** and routes traffic to "unready nodes." Clarification:
- **Launch apps run on AWS Lambda.** There are **no long-lived nodes** — execution environments are **short-lived and fully managed by AWS**. So there's **no node-level readiness/liveness for Launch to probe**.
- **Liveness:** **AWS automatically retires/replaces unhealthy environments** — explicit platform probes aren't needed.
- **Traffic routing:** Lambda has **no "switch traffic to a new node" model**; AWS **only routes to initialized environments**, so unready instances don't receive traffic.
- **Application readiness** is the part Launch handles via the **port-4000 check** (analogous to a container readiness check). If the app can't initialize (e.g. can't reach a dependency), it **should fail fast → not bind 4000 → Launch restarts it.**
- **No health-check-before-routing today.** To add that safeguard yourself: use a **separate Launch environment on the same Git branch/config**, add a **`/health` endpoint**, and have **CI/CD call it and only proceed if it responds**. Also, **deployment halts automatically if the build fails** — extend with **custom build-step checks** (verify CMS/backend connectivity) to gate go-live.

## The real trigger — app fails on `*.contentstackapps.com` but works on the real domain

- **Key finding:** the errors reproduced **only via the default `*.contentstackapps.com` domain**, not via the real Philips domains (`dev.2.philips.ie`, etc.). That points to **request-handling differences by host**, not a platform fault:
  - **Server Actions host check:** logs showed `x-forwarded-host` = `…contentstackapps.com` **does not match `origin`** = `acc.2.philips.ie` → **Next.js aborts the Server Action** (`Invalid Server Actions request`), which preceded the "Address already in use." This is Next.js **Server Actions origin/host validation** failing when accessed via the default domain (headers added by **Akamai** on the real-domain path differ).
  - Recommendation: **block direct access via `*.contentstackapps.com`** (see the default-domain-block entries) so traffic only comes through the properly-headed real domain.
- Also present: **`MODULE_NOT_FOUND`** and other **unhandled errors** that can crash the server if not defensively handled. The **`next-intl` `alternateLinks` `TypeError: Cannot read properties of undefined`** was another app-side error surfaced via the default domain.

## Diagnosing a specific failed request

- The customer's failing response carried **`x-deployment-uid`** and **`x-environment-uid`** headers (plus `x-amzn-requestid`) — **use those to locate the exact environment/deployment instance in logs.**

## Generalized guidance (for future similar queries)

- **"How does Launch health-check my app / recover it?"** → **port-4000 check** (restart if not bound) + **Pingdom** (prod alerting). Recovery = **restart**; a **restart loop** = the app **starts but crashes/again fails** (unhandled error) — **fix in app code** (fail fast on missing deps, handle errors). Launch runs on **AWS Lambda**: **no long-lived nodes, no node readiness/liveness probes, no "switch traffic to node"** — AWS only routes to initialized envs and retires unhealthy ones.
- **No pre-routing health gate natively** → add one yourself: **separate env on same branch + `/health` endpoint + CI/CD gate**, and **custom build-step checks** (deploy halts on build failure).
- **App works on the real domain but 500s / CF001 on `*.contentstackapps.com`** → suspect **host/header-dependent logic**: **Next.js Server Actions `x-forwarded-host` ≠ `origin`** (aborts the action), or logic relying on **Akamai-added headers** present only on the real domain. Fix = **block direct default-domain access** and handle errors defensively.
- **CF001 / "server failed to start" is usually an app crash**, not a platform outage — correlate via **`x-deployment-uid` / `x-environment-uid`**; separate **long-standing app errors** from the real trigger.
- Related: [failed to start server / PORT / EADDRINUSE port 4000 / `document not defined`](../deployments-builds/launch-failed-to-start-server-port-env-var-eaddrinuse-document-not-defined.md), [CF001 on cache miss = app crash / unhandled error → restart mechanism](launch-cf001-on-cache-miss-app-crash-unhandled-error-restart-mechanism.md), [CF001 server failed to start = resource saturation / memory leak](launch-cf001-server-failed-to-start-resource-saturation-memory-leak.md), [block default `*.contentstackapps.com` domain (edge function)](../edge-functions-rewrites/launch-default-domain-indexed-by-google-block-default-domain-edge-function-seo.md), [CloudFront-over-Launch → block default domain via shared header](../edge-functions-rewrites/launch-edge-redirect-loop-cloudfront-origin-block-via-header-authorization.md), [load testing must be pre-approved](launch-load-performance-testing-allowed-must-be-preapproved-checklist.md).

## Status

- **In progress (CL-2033).** **Health model:** **port-4000 check + Pingdom**; recovery = **restart**; on **AWS Lambda** there are **no long-lived nodes / node readiness-liveness probes / node-switch routing** (AWS routes only to initialized envs, retires unhealthy ones). **No pre-routing health gate natively** → suggested **separate env + `/health` + CI/CD gate** and **custom build checks**. **Root trigger = app-side, host-dependent:** fails on **`*.contentstackapps.com`** (Next.js **Server Actions `x-forwarded-host` ≠ `origin`** abort; missing Akamai headers) + **`MODULE_NOT_FOUND`/unhandled errors/`next-intl alternateLinks`** crashes → **block default-domain access + handle errors + fail fast**. Diagnose specific failures via **`x-deployment-uid`/`x-environment-uid`**. Logging friction (gRPC Log Target hard to configure) noted → **HTTP Log Target** under consideration; interim: forward gRPC logs to their **ElasticSearch**. (One internal mix-up: a wrong app's stack traces were shared, then corrected.)
