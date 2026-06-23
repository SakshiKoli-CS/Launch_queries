QUESTION

Customer **Remote.com** (Case **00059455**) hit a Launch **deployment failure on Staging** with **no clear error in the logs**. The build is marked **Failed**, but the deployment log shows **no explicit error at the end** — it appears to **stop mid-process without completing**, inconsistent with a normal successful build.

- **Project:** website-remot — `698dc576a23c100ce055ce94`
- **Environment ID:** `6a22c852c8eadae4514ab53e`
- **Deployment ID:** `6a3011f3ac066a75b1efa53f`
- **Region:** EU
- **Environment:** Staging

ANSWER

**Symptoms**
- Build marked **Failed**; deployment log **truncated mid-process** with **no explicit error** line at the end.

**Investigation**
- Asked the customer to **refresh the screen** and share the **entire deployment logs** (not a screenshot) for full context.
- A build that stops mid-process without a terminal error is consistent with the build **machine running out of resources** rather than an application/code error.

**Cause**
- Customer was **not provisioned with an L2 build machine**. Per the docs, **Enterprise customers should get L2 by default** — see *Build Machines on Launch → Default tiers by plan*: https://www.contentstack.com/docs/developers/launch/build-machines-on-launch#default-tiers-by-plan
- The smaller build machine was the likely reason the build halted mid-process.

**Fix**
- Enable the **L2 build machine** plan key for the customer:
  - **Plan Key:** `contentfly_deployment_build_machine_l2`
  - **Limit:** 1
  - **Max Limit:** 1
- Have the customer **re-deploy** once the plan key is enabled.

**Resolution**
- Plan key **enabled**; customer **re-deployed** and the deployment **succeeded — confirmed resolved**.

**Follow-up / suggested checks**
- For Enterprise customers, verify they are **provisioned with L2 by default**; a build that "fails with no error / stops mid-log" is a strong signal of an **under-provisioned build machine** rather than a code failure.
