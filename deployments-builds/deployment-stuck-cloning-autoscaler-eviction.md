QUESTION

**UVA Health System** (SF **00056397**, project `6800171649412f0653c0eb70`, Azure NA) reported unusually long deployment times after merging Stage → Main and triggering builds via GitHub. Most of their 6 environments were slower than usual, but the **Childrens Main** environment was stuck at the **"Cloning repository"** step for over 20 minutes and took approximately **38 minutes** to proceed.

Characteristics:
- Deploy logs were **blank or only partially populated** during the delay
- **No option to cancel or re-trigger** the stuck deployment
- No recent changes to repository size, dependencies, build config, submodules, or commits
- A similar deployment (Develop → Stage) the previous day completed without issues
- Occurred across multiple environments, most severely on one

ANSWER

**Root cause — cluster autoscaler evicting active deployments during node scale-down**

An edge case in the Launch deployment scheduler caused the **cluster autoscaler to unexpectedly evict ongoing deployments during node scale-down**. This interrupted the cloning step, leaving the deployment stuck with no progress, blank logs, and no cancel/re-trigger option in the UI.

This is a **platform-side incident** — no changes on the customer side caused or could have prevented it.

**Fix**

Safeguards were added to ensure **active deployments are not evicted during autoscaling**. Fix deployed to production. After the fix, customers were asked to redeploy, and subsequent deployments completed normally.

**How to distinguish from DB replication latency (another stuck-deployment cause)**

| | Autoscaler eviction (this entry) | DB replication latency |
|---|---|---|
| Stuck at | "Cloning repository" | Queued / not starting |
| Deploy logs | Blank or partial | Typically absent (deployment never starts) |
| Cancel option | Unavailable | May also fail |
| Fix | Platform safeguard (permanent) | Re-trigger after incident clears |

See also: [Deployment stuck/queued → DB replication latency](launch-deployment-stuck-queued-db-replication-latency.md)

**Resolution**

Platform fix deployed. Customer confirmed no further issues; case closed.
