QUESTION

Customer **UVA Health** (SF case **00057364**, Org `bltdcc012133cba1c83`, Azure NA, `www.uvahealth.com`) reported **intermittent production outages** — a mix of **Connection Timeouts, 500 Internal Server Errors, 404s**, and the error **`Internal Server Error CF001. Server failed to start. Please check Launch Server Logs`** (mobile showed "Launch Server failed to restart"). **No deployments** during the outage windows; recurring frequently (confirmed via UptimeRobot exports). They asked whether there were hosting/infra issues, pod/container restarts, or platform incidents.

_Keywords: CF001, server failed to start, intermittent outage, 500 timeout 404, CPU memory 100%, resource saturation, memory leak, Astro server instances, force restart, increase resources, Kyruus 400, module.page is not a function, Astro static preset, cache priming, no deployment outage._

ANSWER

## Root cause — application resource saturation (CPU/memory hitting 100%)

- The app's **CPU and memory were reaching 100%**. When saturated, **Launch couldn't get timely responses and failed to start new Astro server instances** on demand → **CF001 "Server failed to start"** (surfacing as timeouts/500s).
- Contributing factors in the app:
  - Runtime exceptions during rendering: **`[ERROR] TypeError: module.page is not a function`** (hit on 404 pages).
  - **External API failures (Kyruus `400`)** triggering **retries**, amplifying load.
- The pattern (usage gradually climbing back to 100%) is consistent with a **memory leak / unbounded resource usage** — **not** a platform infra incident, and **no deployments** were involved.

## Mitigations applied (temporary, platform-side)

1. **Increased resources** 1 vCPU → **2 vCPU**, 2 GiB → **4 GiB**. Stabilized briefly, but the app **climbed back to ~100%** even at 2×.
2. **Hourly force restarts** (stop old containers / start new) to **reset memory to baseline** every hour. This held the failures off. **Note:** force restarts are **not typically required** for most Launch apps — it's a stopgap for this app's resource behavior.

## Real fix — application-side (the durable resolution)

Recommended (and what resolved it):
- **Fix memory leaks / unbounded resource usage** in the app.
- **Fix the external Kyruus API errors** (400s) that trigger retries and amplify load.
- Fix the **404 `module.page is not a function`** path (the customer changed Kyruus 404s to redirect to search tabs instead of rendering the broken static 404).
- **Add caching headers** on heavy pages; make high-traffic pages **static** (e.g. `/search`) to cut server load.
- **Use the Astro framework preset in static mode** (no server command) since the site is statically built; add **cache priming**. The Astro preset lets Launch optimize the framework and improves performance. (Astro on Launch docs.)

After the customer deployed these, **resource usage stabilized**, the **hourly force restarts were removed**, and **resources were reverted to default (1 vCPU / 2 GiB)**. Customer **confirmed stability**.

## Clarifications that came up

- **Default allocation (1 vCPU / 2 GiB) is the standard/optimal Launch allocation** — designed to handle significantly higher traffic. The weekend increase was a **response to high utilization**, **not** because the default was "too low." (Don't let customers read the bump as "Launch under-provisioned us.")
- **Server resources are visible in deployment logs (Step 4)** — `[SERVER MACHINE RESOURCES] CPU: … · Memory: …`.
- **Restart/short-term metrics aren't retained** (overwritten each deployment cycle); **server logs retained only 24h** — configure **Log Targets** to stream logs to an external platform for real-time/historical analysis.

## Generalized guidance (for future similar queries)

- **"Intermittent timeouts/500s + `CF001 Server failed to start`, no deployment"** → almost always **app resource saturation** (CPU/mem → 100%) preventing Launch from starting server instances; look for **memory leaks**, **retry storms from failing external APIs**, and **render-time exceptions**. Not a platform incident.
- **Bumping resources is a stopgap** — if usage climbs back to the new ceiling, it's an **app-side leak/inefficiency**; force restarts can hold it temporarily but the fix is in the code.
- **For statically-built Astro sites**, use the **Astro static preset (no server command)** + **cache priming** + caching headers + static heavy pages — dramatically cuts server load.
- **Set expectations:** default resource allocation is standard and handles high traffic; a temporary increase ≠ admission of under-provisioning.
- **Logs:** 24h retention; use **Log Targets** for export/real-time. Resource allocation shows in **deploy log Step 4**.

## Status

- **Resolved** — after app-side fixes (memory/Kyruus/static pages/caching), usage stabilized; **force restarts removed**, resources **reverted to default**; customer confirmed stable. Customer asked to be kept in the loop on any server/memory actions.
