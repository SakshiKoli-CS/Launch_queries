QUESTION

Customer (org **S.E. Demo 2**, Project `691c97907ecdc6639dd4c3e6`, Env `691c97907ecdc6639dd4c3ed`, Deployment `699d891482e615024287d14e`) reports **all deployments failing on Launch**, even though:

- The **build completes successfully on their local environment** with no issues.
- **Redeploying an earlier commit that previously deployed successfully on Launch also fails.**

This is a **blocker for sales demo environments in Europe**. What's causing it?

_Keywords: deployments failing on Launch, builds succeed locally, redeploy old successful commit fails, not code related, platform-side bug, fix released to production, demo environment blocker, deployment failure all commits._

ANSWER

## Cause — platform-side issue (not the customer's code)

- The deployments were failing due to a **platform-side issue on Launch**, **not** the customer's code or recent changes.
- The strong tell: **builds pass locally** and **redeploying a previously successful commit also fails** — when even a known-good commit won't deploy, the problem is on the platform, not the repo.
- The team also reviewed the **commit/deployment history** to rule out a specific change while investigating.

## Resolution — fix released to production

- The Launch team **identified the root cause** and **released a fix to production**.
- After the fix went live, the customer confirmed **deployments are working again**.

## Generalized guidance (for future similar queries)

- **"Deployments failing on Launch, but build succeeds locally AND an old known-good commit also fails to deploy"** → treat as **platform-side**, not code. Don't send the customer chasing their repo first.
- Collect **Project ID / Environment ID / Deployment ID** (all parseable from the deployment URL: `…/projects/<projectId>/envs/<envId>/deployments/<deploymentId>`) and **escalate for investigation**; resolution here was a **platform fix release**, not a customer change.
- Distinct from the **container-app-limit** case (also "build succeeds, old commits fail") — same symptom, different root cause; verify whether it's a **capacity limit** vs a **platform bug needing a fix release**. See [Cloud Functions deploy error / container app limit](launch-cloud-functions-deployment-error-container-app-limit-reached.md).

## Status

- **Resolved.** Root cause was a **platform-side issue**; a **fix was released to production** and the customer confirmed deployments succeed again.
