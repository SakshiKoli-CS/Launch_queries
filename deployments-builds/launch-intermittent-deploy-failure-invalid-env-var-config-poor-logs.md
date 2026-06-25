QUESTION

Customer (Oskar; Project `698469f272a4df27e1043785`, Env `698469f272a4df27e104378d`, AWS EU) hit a **deployment failure** that **resolved itself after a few retries** — they "messed with settings back and forth," **waited ~10 minutes**, and it **worked on the 3rd/4th try with the same settings**. The **same commit had previously deployed successfully**. They asked whether something was **wrong in the AWS EU region**, and noted the **deploy logs weren't informative**.

_Keywords: intermittent deployment failure, worked on retry, same commit deployed before, AWS EU region issue, invalid environment variable configuration, logs not helpful, redeploy after waiting, deployment succeeded after correcting env var._

ANSWER

## Cause — invalid environment variable configuration (not a regional outage)

- Log review showed the **deployment failures were caused by an invalid environment variable configuration.** **After it was corrected, the deployment completed successfully.**
- **Not an AWS EU regional issue** — the region was fine; the failure was **config-specific to this project's env vars.**
- The "worked on the 3rd/4th try with the same settings" feeling lines up with the env-var config being **fixed/normalized during the back-and-forth** (the same commit had deployed before, so the code wasn't the variable — the env config was).

## Note — log quality

- The customer's fair point: the **deploy logs didn't clearly indicate the bad env var.** Better surfacing of **env-var validation errors** in the deployment logs would let customers self-diagnose this class of failure.

## Generalized guidance (for future similar queries)

- **"Deploy intermittently fails then succeeds on retry; same commit deployed before; is the region down?"** → check the **environment variable configuration** first. An **invalid env var** can fail a deploy even when the **code/commit is unchanged and previously worked** — and toggling settings during retries can inadvertently correct it, making it look flaky/regional.
- **A previously-successful commit failing is NOT proof of a platform/region outage** — env-var (and other per-environment config) differences are a common non-code cause.
- **Logs may not name the bad env var clearly** — when a deploy fails without an obvious cause, **audit env vars** (typos, malformed values, missing/duplicate keys) as part of triage.
- Related: [deploy fails on env var with unmatched quotes (parsing)](launch-deployment-failed-env-var-unmatched-quotes-parsing.md), [env var 100-key limit](launch-environment-variable-limit-100-consolidate-workaround.md), [deployments failing — platform bug](launch-deployments-failing-old-commits-too-platform-bug-fix-release.md), [Cloud Functions deploy error — container app limit](launch-cloud-functions-deployment-error-container-app-limit-reached.md).

## Status

- **Resolved.** Cause: **invalid environment variable configuration** (corrected → deploy succeeded); **not** an AWS EU regional problem. Has **not recurred.** Customer flagged that **clearer log hints** would help next time.
