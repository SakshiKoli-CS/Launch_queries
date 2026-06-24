QUESTION

Customer **Philips** (SF case **00056941**, Stack `blte23630023bb36e19`) deploying a **Next.js app from an Azure pipeline via the Contentstack CLI Launch plugin** — the deploy **fails with a 504 timeout**:

```
ApolloError: "The request took too long to process. Please try again later."
extensions: { code: "launch.DEPLOYMENT.REQUEST_TIMEOUT", status: 504 }
```

Consistent on retries (the deploy script ran ~37s on the Azure side before failing). Notably, the **same script/pipeline works on their DEV environment** but fails on **TST**.

_Keywords: launch.DEPLOYMENT.REQUEST_TIMEOUT, 504 deployment timeout, CLI deploy fails but goes live, deployment timeout threshold 14 seconds, false failure deploy, cli-launch plugin, Azure pipeline deploy, ApolloError request took too long._

ANSWER

## Root cause — a ~14s API timeout threshold; the deployment still succeeds

- The **504 `launch.DEPLOYMENT.REQUEST_TIMEOUT`** is triggered by a **timeout threshold (~14 seconds)** on the **deployment API request**. If the deployment request takes **longer than ~14s**, the API returns a **504 timeout response**.
- **But the deployment itself still completes successfully and the site goes live.** So this is a **false failure** — the **CLI/API reports a 504**, while the **deployment is actually fine**. It does **not** affect the deployment or the production website.
- Why **DEV worked but TST didn't**: the deploy request on DEV simply completed **under** the ~14s threshold; the longer TST request crossed it and got the 504 — same script/pipeline, different timing.

## Red herring — it was NOT a Node version issue

- The customer initially suspected **Node.js 20.x vs 22.x** incompatibility (it "worked on 22"), but the **error recurred on Node 22.22.2** too — so the Node version was **not** the cause. (Launch supports Node 20.x.) The real cause was the **API timeout threshold**.

## How to confirm it's the false-positive

- After a CLI 504, **check the Launch UI**: if the **deployment went live** despite the CLI error, you've hit this timeout-vs-actual-success behavior. The deploy succeeding in the UI is the tell.

## Fix / status

- Tracked as **CL-3503**; a **timeout-threshold change was rolled out to production**. (The customer still reproduced it after an initial rollout, so the fix was being **iterated** — confirm current behavior.)

## Generalized guidance (for future similar queries)

- **CLI deploy returns `504 launch.DEPLOYMENT.REQUEST_TIMEOUT`** → likely the **API timeout (~14s) firing while the deployment actually completes**. **Verify in the Launch UI** whether the site went live before treating it as a real failure — it's often a **false failure**, not a broken deploy.
- **"Works on env A, fails on env B with the same pipeline"** → consistent with a **timing/threshold** issue (one request finishes under the limit, the other doesn't), not a config/code difference.
- **Don't chase Node version** for this 504 — it's the request-timeout threshold, not a CLI/Node incompatibility (Node 20.x is supported).

## Status

- Root cause: **~14s deployment-API timeout** returning 504 while the **deploy succeeds** (false failure). Threshold fix rolled out (**CL-3503**) and iterated; verify the current state with the customer.
