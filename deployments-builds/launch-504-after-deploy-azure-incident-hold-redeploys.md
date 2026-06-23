QUESTION

Customer **Concord** (SF case **00058968**) reported that their Launch environment returned a **504 Gateway Timeout following a deployment**. After deploying a minor change, the **deployment was marked successful**, but the environment then became **unavailable with a 504**. The customer tried a **redeployment** and a **rollback to a known-working commit**, but the **issue persisted**.

- **Region:** Azure NA
- **Org ID:** `blt4781beb962759a43`
- **Project ID:** `67b744cb0424db6a7fd21eba`
- **Environment ID:** `67b744cc0424db6a7fd21ec1` (development)
- **Latest Deployment ID:** `6a19769f9d5b5b5c237e348d`

_Keywords: 504 Gateway Timeout after deploy, environment unavailable, redeploy doesn't help, rollback doesn't help, Azure NA US West 2 incident, power event, hold redeployments, manual restore, disaster recovery mechanism, deployment during cloud outage._

ANSWER

## Cause — Azure NA (US West 2) regional incident, not the deployment

- The 504 was caused by an **ongoing Azure NA US West 2 regional incident** (a **power event**), **not** the customer's code change. That's why **redeploy and rollback didn't help** — the failure was upstream infrastructure, independent of the deployed commit.
- Reference: Microsoft Azure status (`https://azure.status.microsoft`). Related Launch incident pattern: [launch-downtime-no-server-logs-azure-regional-incident.md](../logs-monitoring/launch-downtime-no-server-logs-azure-regional-incident.md).

## Immediate guidance during the incident

1. **Hold off on redeploying** until the Azure NA incident resolves — redeploying during the outage can **re-trigger the same failure** (a fresh deploy lands on the impacted region).
2. Launch **manually restored** the affected website. Once restored, advise the customer to **access/refresh the site normally and avoid redeployments** until the incident clears.
3. **Redeploying again could re-break it** while the incident is ongoing — so don't redeploy unless necessary.

## Update — disaster recovery mechanism (unblocks redeploys)

- While the Azure incident was still ongoing with no recovery signal, Launch **implemented a disaster-recovery mechanism** that **allowed customers to resume redeploying**.
- After that, the customer could be told they are **unblocked to redeploy if required**, even with the Azure incident still in progress.

## Generalized guidance (for future similar queries)

- **"504 right after a successful deploy; redeploy & rollback don't fix it"** → strong signal the cause is **upstream/regional infrastructure**, not the code (rollback would fix a code issue). Check the **cloud provider's status** (Azure/AWS) and the Contentstack Status page.
- **During an active regional incident:** advise the customer to **stop redeploying** (a new deploy lands back on the impacted region); Launch may **manually restore** the running site in the meantime.
- **If Launch enables a DR/failover mechanism** during the incident, customers can be **unblocked to redeploy** even before the provider fully recovers — confirm before telling them to redeploy.
- A **"successful" deployment status** does not guarantee the environment is serving — the build can succeed while the serving layer is impacted by an upstream outage.

## Status

- Site **manually restored**; customer advised to hold redeploys during the Azure incident, then **unblocked to redeploy** once the DR mechanism was in place. Root cause: **Azure NA US West 2 regional incident (power event)** — not the deployment.
