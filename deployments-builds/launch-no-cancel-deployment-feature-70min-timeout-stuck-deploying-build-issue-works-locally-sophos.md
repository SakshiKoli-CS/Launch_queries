QUESTION

Customer **Sophos** (SF case **00021834**, Project **Sophos-Dotcom**, Org `blt7eaaa4a78adf9602`, domains `sophos-dev.contentstackapps.com` / `sophos-qa.contentstackapps.com`) reports deployments **stuck on "Deploying" for hours** across multiple attempts and asks Launch to **cancel / "clear" the stuck deployments** so they can do a QA release. No error is shown. The **build runs fine locally**. Can Launch cancel them, and how do they see the build error logs?

_Keywords: clear deployments cancel deployment feature not available, deployment stuck on deploying for hours, no cancel deployments button, build timeout 60 to 70 minutes, multiple deployments each timing out, builds fine locally fails on Launch, check build error logs, add collaborator to inspect build, package dependency build issue, long-running build process._

ANSWER

## There is NO "cancel / clear deployments" feature — deployments auto-timeout at ~60–70 min

- **Users cannot cancel or "clear" a deployment.** There is **no cancel-deployment feature** in Launch.
- A deployment (incl. one with a **long-running build**) **auto-times-out and stops after ~60–70 minutes.** So a deployment that looks "stuck for hours" is really **each attempt individually timing out at ~70 min** (the customer had launched **multiple** deployments; each one hits its own ~70-min timeout and stops). You **wait for the timeout** (or **redeploy**) — there's nothing to cancel from Launch's side.

## Why it looked "stuck for hours" — a long/looping build step

- The customer's build **kept running until the ~70-min timeout** → indicates **their build process runs very long / hangs** (e.g. a step that doesn't terminate). This is a **build-side issue**, not a Launch queue that needs clearing. **Fix the build process and redeploy.**

## "But it builds fine locally" → check the Launch build logs (needs collaborator access)

- **Building locally ≠ building on Launch** — environment/package differences surface only in the **Launch build**. The durable fix here was **package-related** (a dependency issue that manifested only in the Launch build environment).
- To diagnose, **the customer adds the Launch support/eng team as a collaborator on the project** (fastest path) and shares the **project UID**, so the team can inspect the **build error logs** and identify the failing step/package.

## Generalized guidance (for future similar queries)

- **"Cancel / clear our stuck deployments"** → **no cancel feature exists.** Deployments **auto-timeout at ~60–70 min**; **wait it out or redeploy.** "Stuck for hours" across attempts = **multiple deployments each timing out**, not a stuck queue.
- **A deployment that runs to the ~70-min timeout with no error = a long-running/hanging BUILD step** (or a server process started in the build command). Fix the build; don't expect Launch to "clear" it.
- **"Builds fine locally, fails/hangs on Launch"** → inspect the **Launch build logs** (env/package differences don't show locally). **Add the support/eng team as a project collaborator + share the project UID** for the fastest diagnosis. Root cause is frequently a **package/dependency** issue.
- Related: [deployment stuck / hangs indefinitely (server process in build command; 60-min timeout; no cancel)](launch-deployment-stuck.md), [build succeeds, Cloud Functions deploy step fails → container app limit](launch-cloud-functions-deployment-error-container-app-limit-reached.md), [stuck "deploying" 18 min → malformed env vars (quotes/spaces/newlines)](launch-deploy-fail-malformed-env-vars-quotes-spaces-newlines-stuck-deploying-18min.md), [deployment stuck cloning / autoscaler eviction](deployment-stuck-cloning-autoscaler-eviction.md), [deployment stuck queued → DB replication latency](launch-deployment-stuck-queued-db-replication-latency.md).

## Status

- **Resolved.** **No cancel-deployment feature** — the "stuck for hours" deployments were **multiple attempts each timing out at ~60–70 min** due to a **long-running build**; guidance was to **fix the build and redeploy** (nothing to cancel). "Builds fine locally" → **added the team as project collaborator to inspect the Launch build logs**; root cause was a **package/dependency issue**, which the customer **fixed** → case closed.
