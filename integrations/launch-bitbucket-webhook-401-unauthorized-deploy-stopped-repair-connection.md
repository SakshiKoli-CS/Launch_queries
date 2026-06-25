QUESTION

Customer **Conforama** (SF case **00052959**) suddenly **can't deploy with Launch** (since noon Paris time). The **Bitbucket → Launch webhook** (Git provider event) returns **401 Unauthorized**:

```
POST https://gcp-eu-launch-api.contentstack.com/manage/git-provider-events/externalgitprovider?installationId=687a4f96a383af09d0050530
{"errors":[{"code":"Unauthorized","message":"Unauthorized"}],"status":401}
```
Request headers show a genuine **Bitbucket `repo:push` webhook** (`User-Agent: Bitbucket-Webhooks/2.0`, `X-Hub-Signature-256: …`).

- The customer **tried Repair Connection** and **created a new project** — same failure.
- It affects **both GCP-EU and AWS-EU** Launch environments → looks like an **auth problem between Launch and Bitbucket**, not one project.
- They ask: should they **uninstall/reinstall the Bitbucket app**, and is there a **risk to existing environments**?

_Keywords: Bitbucket webhook 401 Unauthorized, cannot deploy, git-provider-events externalgitprovider, repo:push, X-Hub-Signature, auth between Launch and Bitbucket, Repair Connection, uninstall reinstall Bitbucket app risk, commit list empty cannot load commits, intermittent after prod push, GCP-EU AWS-EU._

ANSWER

## Cause — platform-side Launch↔Bitbucket auth issue (not the customer's setup)

- The **401 Unauthorized** on the Bitbucket webhook was a **platform-side issue with the auth between Launch and Bitbucket**, **not** a misconfiguration on the customer's side.
- The tells: **Repair Connection didn't help**, a **brand-new project failed identically**, and it spanned **both GCP-EU and AWS-EU** — so it wasn't a single project/connection.

## Guidance during the incident

- **Hold off on using the Bitbucket Cloud app** and **don't uninstall/reinstall** while the team investigates (avoid risk to existing environments). The fix was applied **platform-side**.
- After the fix: use **Repair Connection** in the Launch UI to re-establish the connection.

## Recurrence — tied to a production push; "commit list empty / can't load commits"

- It **recurred** and was **intermittent** ("working now, not this morning"), including **empty commit list / Launch unable to load commits**.
- This **correlated with a Launch production push** — i.e. the instability tracked the **platform deploy**, not customer changes. Resolved again platform-side.

## Generalized guidance (for future similar queries)

- **"Suddenly can't deploy; Bitbucket (or Git) webhook returns 401 Unauthorized"** → suspect a **platform-side Launch↔Git-provider auth issue**, especially when **Repair Connection fails**, a **new project fails the same way**, and **multiple regions (GCP-EU + AWS-EU)** are affected. Don't have the customer uninstall/reinstall the app (risk + won't help) — **escalate**; the fix is platform-side, then **Repair Connection**.
- **Intermittent Git instability / "commit list empty / can't load commits"** that flips working↔broken can be **correlated to a Launch prod push** — check deploy timing before chasing the customer's repo.
- Distinct from per-customer Git issues: lost-connection w/ permissions ([repair does nothing / project limit](launch-lost-repo-connection-repair-does-nothing-project-limit.md)), GitHub org/SSO approval, branch-sync UI bug — those are config/UI-scoped, whereas this was a **broad auth/region-wide platform fault**.

## Status

- **Resolved (platform-side).** Root cause: a **Launch↔Bitbucket auth issue** causing **401** on the deploy webhook (region-wide GCP-EU/AWS-EU; Repair Connection + new project didn't help). Fixed; **Repair Connection** re-established it. A **recurrence** (intermittent, empty commit list) **coincided with a Launch prod push** and was resolved again.
