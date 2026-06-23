QUESTION

Customer **MicroStrategy** (SF case **00059137**) reported a Launch **build queued for more than 20 minutes** and asked support to **force the deployment to fail** so they could redeploy. They noted this was the **2nd occurrence in ~2 weeks**.

Additional details that came up:
- Attempting **"Cancel deployment"** returned: *"Could not cancel due to a technical error. The deployment is still in progress. Try canceling again."*
- An **urgent deployment that normally completes in minutes took nearly 2 hours.**
- The customer asked whether using the **Launch CLI** for redeployment would have **avoided the delay**, or whether the degradation affected **backend deployment processing** too — so they could prepare an alternative approach for the future.

- **Project ID:** `6765d82c9f1088499e8a2820`
- **Org ID:** `blt45a2a52367b47001`

_Keywords: deployment queued 20 minutes, force fail deployment, cancel deployment failed technical error, "The deployment is still in progress", Launch CLI vs UI, backend deployment processing degraded, recurring stuck deploy, urgent deploy took 2 hours, replication latency._

ANSWER

## Cause — DB-side replication latency (platform incident)

The queued/stuck deployment originated on the **database side**: a **DB migration activity triggered a cleanup process** that **increased data replication latency** due to **high load on the replica database**, causing the deployment performance issues during that window. Transient platform incident, not a project-specific problem. (See related: [launch-deployment-stuck-queued-db-replication-latency.md](launch-deployment-stuck-queued-db-replication-latency.md).)

## "Cancel deployment" failing

- During the incident, **"Cancel deployment" itself can fail** — *"Could not cancel due to a technical error. The deployment is still in progress. Try canceling again."* — consistent with the same backend degradation.
- **Workaround when cancel fails:** navigate to a **previous successful deployment and redeploy it**, then re-trigger the intended deployment once the platform issue clears. (After the fix, re-triggering completed successfully.)

## Production impact — none to traffic, but operations WERE disrupted

- **No impact on production traffic or customer-facing sites** — live sites and end users were unaffected.
- **But** the customer's point stands: the **backend deployment processing itself was degraded**, so an urgent deploy that normally takes minutes took ~2 hours. "No production impact" ≠ "no business impact" — acknowledge the **operational disruption**.

## Launch CLI vs UI — would the CLI have avoided it? → No

- **No.** The degradation affected the **backend deployment processing**, so deployments triggered via **either the Launch UI or the Launch CLI** would have experienced the **same delays**. Switching to the CLI is **not a workaround** for this class of incident.

## Mitigation

- Cause identified; **changes implemented to reduce the likelihood and impact** of similar incidents, with continued monitoring and further deployment-process improvements planned. An RCA was shared.

## Generalized guidance (for future similar queries)

- **"Make my stuck deployment fail so I can redeploy" / "force-fail"** → if **Cancel** errors out, redeploy a **previous successful deployment** instead; the queue typically clears once the platform incident resolves.
- **"Would the Launch CLI be faster / avoid the delay?"** → **No** during a backend-processing incident — UI and CLI hit the same backend; the CLI is not an escape hatch.
- **Recurring stuck deploys ("2nd time in 2 weeks")** → flag as a **platform incident with an RCA**, not a per-project build issue; confirm mitigation has been applied.
- **Be precise about impact:** distinguish **production traffic** (unaffected) from **deployment operations** (degraded) — customers with time-sensitive deploys feel real disruption even when their live site is fine.

## Status

- **Resolved.** Deployment completed after the platform fix; root cause was replica-DB replication latency from a migration cleanup; CLI would not have helped; mitigations implemented to reduce recurrence.
