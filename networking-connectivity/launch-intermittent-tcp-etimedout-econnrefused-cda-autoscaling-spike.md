QUESTION

Customer **Concord / Assurant** (Org `blt4781beb962759a43`, Project "Assurant" `67b744cb0424db6a7fd21eba`, SF case **00054053**) reports **intermittent connectivity failures from their Launch environments to the Contentstack Content Delivery API (CDA)** on 26 Feb 2026.

- **Windows:** 00:09–00:30 UTC (~21 min, **~4,400+ failures across 6 pods**) and 15:52–15:57 UTC (~5 min, **76 failures across 2 pods**).
- **TCP-level connection errors** to CDN endpoints `104.16.115.41:443` and `104.16.116.41:443`: mostly **`ETIMEDOUT`** (connect timeout), some **`ECONNREFUSED`**. **Not HTTP-level** — the app can't establish a TCP connection.
- Errors hit **all pods uniformly** (not a single instance).
- **External API monitoring (from outside the hosting env) showed 100% uptime** in the same windows → suggests the issue is on the **network path between Launch infra and the CDN**, not a general API outage.
- Affected envs: Production / Acceptance / Development. Later: customer still sees **alerts concentrated in DEV and ACCEPT** (DEV is **low-traffic, barely used**), arguing it's **not traffic-related but communication-related** — e.g. **26 alerts in DEV** over a weekend.

_Keywords: intermittent connectivity CDA, ETIMEDOUT ECONNREFUSED, TCP connection errors to CDN 104.16.x.x:443, all pods uniformly, external monitoring 100% uptime, rapid pod scaling spike, aggressive autoscaling, concurrency, load testing not announced, DEV low traffic alerts, 500 errors caused by application._

ANSWER

## No platform-side API outage in the reported windows

- Reviewed the incident: **no platform-level CDA outage** during the reported windows. External monitoring showing **100% uptime** is consistent with that.

## Leading hypothesis — autoscaling spike → TCP-level errors

- The **rapid spike in pod count** suggests a **sudden surge in concurrent outbound requests**. If the **app isn't processing requests fast enough**, it can trigger **aggressive scaling**, which produces **TCP-level errors like `ETIMEDOUT` / `ECONNREFUSED`** on outbound connections (connection setup pressure during a scale storm), uniformly across pods.
- Recommendations: **optimize request handling**, **limit concurrency**, and **tune autoscaling** to avoid rapid scaling spikes.

## Check for unannounced load testing

- Asked whether **load testing** ran in the spike window (1:30–3:00 PM IST). **Load testing must be coordinated with the Launch team beforehand** — see the [load testing doc](https://www.contentstack.com/docs/developers/launch/load-testing).

## Follow-up investigation (DEV/ACCEPT alerts, "communication not traffic")

- Customer pushed back: DEV is **low-traffic / barely used**, yet had **~26 alerts over a weekend** — so they suspect a **communication** issue, not traffic.
- Deeper platform-side review (last 7 days): **no 5xx from the CDA API**, and **no `ETIMEDOUT` / `ECONNREFUSED`** observed platform-side. The **only errors seen were 500s caused by the application**, and **those stopped after 3 March**.
- Asked the customer to **confirm any changes/deployments around that timeframe** (to correlate) and to **add additional debug logging in the application** for further troubleshooting.

## Generalized guidance (for future similar queries)

- **TCP-level `ETIMEDOUT` / `ECONNREFUSED` to the CDA/CDN from Launch pods, while external monitoring shows the API is up** → it's **not a CDA outage**. Two prime causes to separate:
  1. **Autoscaling / concurrency spike** — a surge in concurrent outbound requests + slow request processing triggers aggressive scaling and connection-setup failures across all pods. Fix on the app side: **cap concurrency, optimize handlers, tune autoscaling.**
  2. **Unannounced load testing** — coordinate load tests with Launch first (per the load-testing doc); unannounced bursts look exactly like this.
- **"Low-traffic env (DEV) still alerting"** doesn't automatically mean a platform comms problem — verify platform-side first (look for 5xx / TCP errors). Here the platform side showed **only application 500s** that **stopped after a date**, so the next step is to **correlate with the customer's deployments** and add **app-level debug logging**.
- **External-uptime vs internal-failure mismatch** is the key diagnostic: it points at the **app/egress path under load**, not the API.

## Status

- **No platform API outage** confirmed for the reported windows. Recommended **concurrency/autoscaling tuning** + **announcing load tests**. On follow-up, platform side saw **only application 500s (stopped after 3 Mar)** — asked customer to **correlate with their changes** and add debug logging. Customer planned to deploy changes in their **normal release cycle**; **awaiting post-go-live confirmation** (open/monitoring).
