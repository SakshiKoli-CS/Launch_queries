QUESTION

On a **non-production environment** (`dev22` / `csnonprod`, Stack `blt8a1ab9a67d4f144b`, Org `blt085660ee5f1dd33a`), a **Launch deployment failed when SSO was enabled**, but **worked when SSO was disabled**.

_Keywords: SSO enabled deployment fails, SSO disabled works, Launch deploy fails with SSO, non-production environment, dev22 csnonprod, redeploy fixes, transient deploy failure._

ANSWER

## Observation

- **SSO enabled → deployment fails; SSO disabled → deployment works** — observed on an **internal non-production** environment (`dev22`), where behavior can be unstable during active development.

## Resolution

- **Re-deploying** resolved it — the deployment succeeded on retry. The failure appears to have been **transient**, not a persistent SSO-vs-deploy incompatibility.

## Notes / labeled uncertainty

- The thread did **not** establish a confirmed root cause for why SSO-enabled deploys failed at that moment; it cleared on a redeploy. Treat as a **transient failure on a non-prod environment** rather than a documented SSO limitation.
- For **production-grade stability**, use a **production environment** — non-prod/dev environments can break during development.

## Generalized guidance (for future similar queries)

- **"Launch deployment fails with SSO enabled (works with SSO off)"** → first **retry the deployment**; if it was transient (esp. on a non-prod env), it'll succeed. Don't assume a hard SSO/deploy incompatibility without it being reproducible.
- **If it reproduces consistently**, escalate with the env/deployment IDs and capture deploy logs — but in this case a **redeploy** was sufficient.
- **Non-prod environments are expected to be less stable**; for reliability use **production** environments.

## Status

- **Resolved** via **redeploy** (transient failure on a non-prod env). No confirmed SSO-specific root cause documented.
