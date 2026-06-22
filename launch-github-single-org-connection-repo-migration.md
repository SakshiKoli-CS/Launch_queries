QUESTION

Customer **Eight25Media** (SF case **00059380**) is evaluating migrating a Launch project from a repository under **GitHub Organization A** to a repository under **GitHub Organization B**. They have multiple Launch projects connected to repos in Org A, created a mirrored repo in Org B, and are testing with a non-production Launch project.

During testing they found Launch appears to allow **only a single GitHub account/organization connection at a time** — the only path seems to be **disconnecting the existing GitHub integration before connecting the new one**, which they fear will break the other projects still using Org A.

**Customer questions:**
1. Is Launch limited to a **single GitHub account/organization connection** per Launch account?
2. Can **multiple GitHub orgs/accounts** be connected simultaneously?
3. Can an existing Launch project + environments be **reconnected to a different repo under another org**?
4. What is the **recommended migration path** from GitHub Org A → Org B?
5. Can this be done **without impacting existing projects** that rely on the current GitHub integration?
6. Are there **planned enhancements** to support multiple GitHub orgs/accounts in the same Launch account?

_Keywords: GitHub integration, single organization connection, multiple GitHub orgs, disconnect GitHub, reconnect repository, change repo, org A to org B migration, source repository migration, Launch GitHub limit, roadmap multiple orgs._

ANSWER

## Current behavior (confirmed)

- **Yes — Launch supports a single GitHub account/organization connection at a time** per Launch account. Multiple GitHub orgs/accounts **cannot** be connected simultaneously today.
- To switch to **Organization B**, you **disconnect Organization A first, then connect Organization B.**

## Key reassurance — disconnecting does NOT break running projects

- **Disconnecting Org A will not affect currently deployed projects running on Org A.** Existing deployments and live environments **continue to function as-is.**
- So the switch can be performed **seamlessly without disruption** to running projects — the GitHub connection governs new connect/deploy actions, not the continued operation of already-deployed environments.

## Recommended migration path (Org A → Org B)

1. Have the **mirrored repository ready in Org B** (customer already did this).
2. **Disconnect** the existing **GitHub Organization A** integration in Launch.
3. **Connect GitHub Organization B.**
4. Point the target Launch project/environments at the **Org B repository**.
5. Existing Org A–based projects keep running throughout — no redeploy or disruption needed for them.

This migration path works exactly as expected.

## Roadmap

- Support for **multiple GitHub organizations/accounts within the same Launch account** is **on the roadmap** as a planned enhancement. More details to follow as it progresses.

## Resolution

- Explanation **addressed all customer questions**; customer confirmed **no further assistance needed** and the case was **resolved and closed.**

## Generalized guidance (for future similar queries)

- **"Can I connect more than one GitHub org to Launch?"** → Not currently; **one GitHub org/account connection at a time**. Multi-org support is **on the roadmap**.
- **"Will disconnecting GitHub break my live sites?"** → **No.** Already-deployed projects and live environments keep running; the connection only affects new connect/deploy operations.
- **"How do I move a project to a repo in another GitHub org?"** → Disconnect the current org, connect the new org, repoint the project/environments to the new repo. Other existing projects are unaffected.
