QUESTION

Customer reports a **deployment failure in their development environment** on Launch (project `66fdaa41372b77487cd6afa3`, env `66fdaa41372b77487cd6afaa`, Azure NA). The **build completes successfully**, but it **fails at the Cloud Functions deployment step**:

> "Cloud functions deployment error: Please consult the Cloud Functions documentation for troubleshooting steps, or contact support for assistance. Deployment failed: Please try to redeploy the site, or contact support for assistance."

Key signals that it's **not code-related**:
- The failure **also occurs when redeploying a previously successful deployment.**
- They **reverted edge function changes**, added alternative code, **reverted to a previously deployed commit**, and **force-pushed** — **still** fails with the same **Cloud functions deployment error**.
- **Production site is running fine**; they're only trying to deploy to **development** (to add logging for unusually high redirect logs from Next.js middleware).

_Keywords: Cloud functions deployment error, build succeeds deploy fails, redeploy previously successful fails, reverted commit force push still fails, container app limit reached, Azure, development environment deployment failure, nothing deploys to environment._

ANSWER

## Cause — container app limit reached (on Production)

- The deployment failed because the **container app limit was reached on Production**. Even though the customer was deploying to **development**, the **production-side limit** blocked new Cloud Functions deployments.
- This is a **platform-side capacity limit**, **not** the customer's code — which is exactly why **redeploying old successful commits / reverting / force-pushing all kept failing** with the same Cloud functions deployment error.

## Resolution

- The team **addressed the container app limit on Production**; the issue resolved.
- Asked the customer to **redeploy** and confirm the deployment completes successfully.

## Generalized guidance (for future similar queries)

- **"Build succeeds but Cloud Functions deployment step fails"**, **and** redeploying a known-good commit / reverting / force-pushing **all fail the same way** → strong signal it's **platform-side, not code**. Prime suspect: **container app limit reached** (capacity), which can be hit at the **Production** level and block deployments to other environments too.
- The generic **"redeploy the site, or contact support"** Cloud Functions error is **not diagnostic on its own** — when it's reproducible across reverts/force-pushes, **escalate to check the container app limit / capacity** rather than chasing code changes.
- **Symptom shortcut:** "nothing deploys to that environment anymore, even old commits" + Cloud functions deployment error = capacity/limit, not the repo.

## Status

- **Resolved.** Cause was the **container app limit reached on Production**; team raised/addressed it. Customer asked to **redeploy and confirm** success.
