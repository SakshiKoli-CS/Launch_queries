QUESTION

Customer **Sophos** (SF case **00058612**) reported that **newly created branches following a `release-*` naming convention are not appearing in the Launch UI branch-selection dropdown** across **all environments** (PRODUCTION, DEMO, STAGING, QA, DEV).

Key observations isolating the bug:
- **Other prefixes created at the same time (e.g. `hotfix/*`) sync instantly** and appear correctly.
- **The Launch API recognizes and switches to the missing `release-*` branches** — so the backend / Git integration *is* aware of them; only the **UI dropdown state fails to update**.
- **Missing in UI:** `release-20260519-a`, `release-20260519-b`, `release-20260520-a`.
- **Visible (expected):** older release branches (`release-20260415-a`, `release-20260423-a`) and new non-release branches (`hotfix/*`).
- **Workaround:** customer switched branches directly via the **Launch API endpoints**, bypassing the UI.
- Project: `sophos-dotcom`, Org `blt7eaaa4a78adf9602`, Git provider **GitHub**.

_Keywords: branch not showing in dropdown, branch sync, release-* branches missing UI, GitHub integration, API sees branch but UI doesn't, branch selection dropdown, hotfix syncs but release doesn't, CL-4062._

ANSWER

## Cause — platform bug in UI branch-dropdown sync (not a customer config issue)

- This was a confirmed **Launch platform bug**: certain newly created branches (the `release-*` set here) were **recognized by the backend/Git integration and the Launch API** but **failed to populate in the UI dropdown** state.
- The fact that the **API could see and switch to the branches**, and that **other prefixes (`hotfix/*`) synced fine**, confirmed the backend Git sync was working — the defect was specifically in the **UI dropdown updating**.
- Tracked internally as **CL-4062**.

## Workaround (while the fix was pending)

- **Switch branches via the Launch API endpoints** directly — this works even when the UI dropdown doesn't list the branch.

## Resolution

- The team **identified the issue**, made changes with cross-environment testing, and **deployed the fix to production**. Verified that **branches now sync correctly to the Launch UI dropdown**; customer confirmed the fix works. Case closed.

## Generalized guidance (for future similar queries)

- **"A new branch doesn't show in the Launch UI dropdown"** → first check whether the **API/backend sees it** (e.g. switch via the API). If the **API knows the branch but the UI doesn't**, it's a **UI dropdown sync issue**, not a Git-integration or customer-config problem.
- **Selective by prefix/timing** (e.g. `hotfix/*` syncs but `release-*` doesn't) points to a **platform bug** rather than anything the customer did — escalate with the specific branch names and prefixes.
- **Interim workaround:** drive branch switching through the **Launch API** until the UI sync is fixed.
