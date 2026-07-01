QUESTION

Customer **Cherokee Nation Entertainment (CNE)** and their SI **Huge**, migrating off **Sitecore** (which has downtime notifications), ask before a go-live: **does Launch have monitoring and automated notifications when their sites are down or need maintenance?**

_Keywords: Launch uptime monitoring, automated downtime notification, site down alert, maintenance notification, does Launch notify when site is down, internal monitoring only not external-facing, no customer-facing alert system yet, external monitoring Pingdom Site24x7, migrating off Sitecore notifications, go-live monitoring._

ANSWER

## What Launch provides

- **Launch sets up internal monitoring for all production websites.** This monitoring is **internal-facing** — it is **not exposed to customers** and does **not** send automated external notifications to them.
- **There is no customer-facing automated notification system yet.** If a site goes down, the **Contentstack support team will notify the customer** (manual/human, not automated self-service alerting).

## What the customer can do for their own automated alerts

- **Set up external monitoring themselves** using a third-party uptime service such as **Pingdom** or **Site24x7** (or equivalent — e.g. a synthetic HTTP check on the site's root URL). This gives them **their own automated down/maintenance notifications** independent of Launch.

## Generalized guidance (for future similar queries)

- **"Does Launch alert us when our site is down?"** → **Launch monitors all production sites internally**, but that monitoring is **internal-only**; there is **no automated customer-facing notification system**. Support **notifies the customer manually** on downtime.
- **For automated, customer-owned uptime alerts, recommend an external monitor** (**Pingdom / Site24x7** / any synthetic HTTP check on the root domain). This doubles as a **keep-warm** mechanism for idle containers.
- Set this expectation **before go-live**, especially for customers **migrating off a platform (e.g. Sitecore) that had built-in notifications**.
- Related: [cold starts → keep-warm with Pingdom health check](launch-cold-starts-idle-container-keep-warm-pingdom.md), [downtime with no server logs = regional incident](launch-downtime-no-server-logs-azure-regional-incident.md), [Cloudflare 504 downtime → retry/redeploy/proactive guard](launch-downtime-cloudflare-504-retry-redeploy-proactive-guard.md), [log targets (Datadog) for customer-side observability](log-target-datadog-setup.md).

## Status

- **Answered.** Launch runs **internal monitoring on all production sites** but it is **internal-facing** with **no automated customer-facing notifications** — **support notifies manually** on downtime. For their own automated alerts, the customer can **set up external monitoring (Pingdom, Site24x7)**. Relayed to CNE / Huge ahead of go-live.
