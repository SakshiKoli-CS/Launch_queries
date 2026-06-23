QUESTION

Customer **Horizontal Digital** (SF case **00058960**) reported **504 Gateway Timeout** errors on **both** of their Launch sites:

- `https://srpnet-uat.azcontentstackapps.com/`
- `https://srpnet-dev.azcontentstackapps.com/`

Both sites **were working until yesterday** and the issue **started this morning**; the 504 was verified on both URLs. **No deployment** was implicated — the sites simply went down. Customer wanted an **urgent** update.

- **Region:** Azure North America
- **Org ID:** `blt6d328981cb625d90`
- **Stack API key:** `bltefc1fc708dcc94e7`

_Keywords: 504 Gateway Timeout, bad gateway, sites down, working until yesterday, azcontentstackapps.com, Azure NA US West 2 incident, power event, upstream cloud provider, hold redeploys, manual restore, disaster recovery, status page incident._

ANSWER

## Cause — Azure NA (US West 2) regional incident

- The 504s trace to an **ongoing Azure NA US West 2 regional incident** — a **power event** affecting **Virtual Machines** in that region. **Not** a Launch defect and **not** anything the customer changed (no deploy involved; sites that "worked yesterday" went down).
- **Azure impact statement:** customers using **VMs in West US 2** received errors on service-management operations (create/delete/update/scale/start/stop) for resources in that region.
- References: Contentstack Status incident `https://status.contentstack.com/incidents/57l45wjs31rz` (Azure NA – Proactive Monitoring of Upstream Cloud Provider Issue); Microsoft Azure status `https://azure.status.microsoft`.

## Immediate guidance during the incident

1. **Hold off on redeploying** until the Azure NA incident resolves — a fresh deploy can land back on the impacted region and re-break the site.
2. Launch **manually restored** the affected sites. Once restored, the customer can **access/refresh normally** and **avoid redeployments** until the incident clears.
3. Redeploying during the incident **could re-trigger the failure**.

## Update — disaster recovery mechanism (unblocks redeploys)

- With the incident still ongoing and no Azure recovery signal, Launch **implemented a disaster-recovery mechanism** that **allows customers to resume redeploying**. The customer can be told they are **unblocked to redeploy if required**.

## Generalized guidance (for future similar queries)

- **"My Launch site(s) suddenly return 504 — worked yesterday, no deploy"** → check the **cloud provider status** (Azure/AWS) and the **Contentstack Status page** before investigating the app; widespread 504s with no deploy point to an **upstream regional incident**.
- **During an active regional incident:** advise customers to **stop redeploying** (a new deploy re-lands on the impacted region); Launch may **manually restore** the running sites.
- **If Launch enables a DR/failover mechanism**, customers can be **unblocked to redeploy** before the provider fully recovers — confirm before advising.
- Related, same incident class: [504 after a successful deploy](launch-504-after-deploy-azure-incident-hold-redeploys.md) and [downtime + no server logs](../logs-monitoring/launch-downtime-no-server-logs-azure-regional-incident.md).

## Status

- Sites **manually restored**; customer advised to hold redeploys during the Azure incident, then **unblocked to redeploy** once the DR mechanism was in place. Root cause: **Azure NA US West 2 regional incident (power event)**.
