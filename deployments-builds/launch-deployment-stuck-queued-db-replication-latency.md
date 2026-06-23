QUESTION

Multiple customers reported **deployments stuck / queued** on Launch for several minutes (one report ~20 minutes before completing). Example: "deployment stuck for a few mins, anything I can do about this?" Two+ such reports came in around the same window. Was there a platform issue, and what was the cause?

_Keywords: deployment stuck, deployment queued, stuck in queue, slow deployment, redeploy, DB migration, replication latency, replica database load, Launch UI management operations, no production impact._

ANSWER

## Cause — database-side replication latency (platform incident)

- The stuck/queued deployments originated on the **database side**: a **DB migration activity triggered a cleanup process** that **increased data replication latency** due to **high load on the replica database**, causing the performance issues during that window.
- This was a **transient platform incident**, not a per-project misconfiguration.

## Scope — no production impact

- The impact was **limited to management operations in the Launch UI** (e.g. triggering/queuing deployments).
- **No impact on production traffic or customer-facing sites** — live sites and end users were unaffected. Only the deploy/management plane was slow.

## Resolution

- Once the underlying DB load/replication latency cleared, the issue **resolved**. Customers were asked to **re-trigger the deployment**, which then **completed successfully**. (An RCA was also shared with affected enterprise customers.)

## Generalized guidance (for future similar queries)

- **"My deployment is stuck/queued for several minutes"** → If multiple customers report it in the same window, suspect a **transient platform-side incident** (e.g. DB replication latency from a migration/cleanup), not the customer's project.
- **Immediate action for the customer:** **re-trigger the deployment** once the platform issue clears; queued deploys typically complete on retry.
- **Reassurance:** deploy-queue/Launch-UI slowness of this kind is a **management-plane** issue — it does **not** affect already-deployed production sites or end-user traffic.
- Check for a **platform incident / RCA** before treating a single stuck deployment as a project-specific build problem.

## Status

- **Resolved.** Deployments completed on retry; root cause was replica-DB replication latency from a migration cleanup; no production impact.
