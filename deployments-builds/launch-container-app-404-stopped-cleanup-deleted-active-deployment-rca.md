QUESTION

Customer **PODS** (Org `681a413e8325a372e42f38b2`, Azure NA) reports that **on rare occasions, post-deployment, containers become unresponsive** in their lower environments. Visiting their URLs (e.g. `https://stage.pods.ca/`) returns:

> **"Error 404 - This Container App is stopped or does not exist."**

The same happened on `https://stage.pods.com/` but **cleared after running another deployment**. They want to understand the **RCA** and get ahead of it so it can't happen in production.

_Keywords: Error 404 This Container App is stopped or does not exist, container unresponsive post deployment, Azure Container App deleted, cleanup job deleted active deployment, Current Deployment ID marked archived, redeploy fixes 404, deployment state inconsistency, lower environments stage._

ANSWER

## Root cause — cleanup job deleted the ACTIVE deployment's container

- The **404 ("Container App is stopped or does not exist")** occurred because **active Azure container app resources were unintentionally deleted.**
- Rare edge case: the **Current Deployment ID was incorrectly marked as archived** (an **infrastructure-level inconsistency in deployment state handling**).
- The **automated cleanup job** then **interpreted the active deployment as archived and deleted its live container resources** → 404 responses.
- This is why a **redeploy clears it**: a fresh deployment recreates the container that was wrongly removed.

## Fix

- Added **guard conditions in the cleanup workflow**: it **must not delete** when the **Current Deployment ID and Archived Deployment ID are identical** — ensuring **live deployment resources can't be removed** due to a state inconsistency.

## Preventive measures

- Added **validation logic to protect active deployments** and **safeguards against misclassification of deployment states.**
- Initiated **deeper infrastructure analysis** to trace and eliminate the root state inconsistency.
- **Enhanced monitoring/traceability** for faster detection and response if it recurs (and the new guard prevents deletion of live containers).

## Generalized guidance (for future similar queries)

- **"Error 404 - This Container App is stopped or does not exist" appearing post-deployment (esp. intermittently)** → the **live container was deleted by the cleanup job** after the **active deployment was misclassified as archived**. **Immediate workaround: redeploy** (recreates the container).
- This is **platform-side**, not the customer's app. RCA = **deployment-state inconsistency** (Current Deployment ID flipped to archived) + **cleanup job over-deleting**; the **guard condition** (don't delete when Current == Archived deployment ID) is the durable fix.
- Distinct from a **container app limit / capacity** failure — here the container was **deleted**, not blocked by a quota.

## Status

- **Resolved.** RCA: **cleanup job deleted the active deployment's container** after the **Current Deployment ID was wrongly marked archived**. Fix (guard condition: don't delete when Current == Archived ID) **verified post-deployment in production**; redeploy clears any current 404. Preventive validation + monitoring added.
