QUESTION

Customer **HUGE** (Org `bltf25b07b22aa512b4`, Project "hsip-website" `6888fe93d8181072d8b0e03b`, Azure) reports **urgent intermittent deployment failures** — the **same code randomly fails or succeeds** on deployment attempts. Error:

> "Deployment failed while setting up the site: Please try to redeploy the site, or contact support for assistance."

- Intermittent across **stage and test** environments (`hsip-website-stage`, `hsip-partners-stage`, `hsip-website-test`, `hsip-partners-test` `.azcontentstackapps.com`).
- **Production release scheduled for the next day** — customer blocked by the random failures.
- Logs show the failure occurs after **Cloning repository / Installing dependencies** (Node v22.21.1).

_Keywords: intermittent deployment failure, Deployment failed while setting up the site, same code randomly fails or succeeds, redeploy or contact support, limited to Launch deployment pipeline, live sites unaffected, stage test environments, production go-live blocked, HUGE hsip-website._

ANSWER

## What's known

- The failure is **intermittent** and **limited to the Launch deployment pipeline** (the "setting up the site" stage), with the **same code** succeeding on some attempts and failing on others.
- **Already-live deployments are unaffected:** any deployment that is **already Live (or gets marked Live) continues to serve traffic normally** — the issue is in **getting new deployments through the pipeline**, not in serving.
- Not attributable to the customer's code (same commit flips pass/fail), consistent with a **platform-side pipeline / transient infra issue** (cf. other intermittent deploy failures that turned out to be platform/network, not code).

## What to tell the customer (reassurance + safety)

- It's **safe regarding live traffic** — Live sites keep serving. The risk is only that a **new deployment may need re-attempts** to complete.
- For a go-live: **retry the deploy** until it completes/marks Live; once Live it's stable. (Customer was able to **successfully take 2 websites live** this way while investigation continued.)

## Generalized guidance (for future similar queries)

- **"Deployment failed while setting up the site" / intermittent, same code passes-then-fails** → treat as a **Launch pipeline / transient issue**, not the customer's code. **Live deployments keep serving**, so it's not a traffic-down emergency — the impact is **blocked/unreliable new deploys**.
- **Mitigation:** **redeploy/retry** to get through; **escalate** for the pipeline investigation (collect Org/Project UID, affected env URLs, deployment logs).
- Distinguish from non-platform intermittent causes already cataloged: [invalid env var config](launch-intermittent-deploy-failure-invalid-env-var-config-poor-logs.md), [electron/large-binary install glitch](launch-uva-frequent-deploy-failures-ssg-rebuilds-network-electron-ssr-recommendation.md), [sst install $USER](launch-sst-install-build-failure-user-cgo-error-user-root-avoid-sst-in-build.md), and platform-bug [deployments failing — fix released](launch-deployments-failing-old-commits-too-platform-bug-fix-release.md). When logs show no app-side cause and it flips pass/fail, lean platform-side.

## Status

- **Open / under investigation** (no confirmed root cause in-thread). Confirmed: **intermittent**, **limited to the Launch deployment pipeline**, **Live sites unaffected**. Customer **went live with 2 sites** via retries; team **actively investigating**, updates pending. (Labeled uncertainty: root cause not yet established.)
