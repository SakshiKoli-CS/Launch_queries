QUESTION

Customer **Motorola** got an error when trying to **connect Launch to an SSO-protected GitHub Org**. (Trial org — their infra isn't provisioned yet.) Is there guidance for connecting Launch to a GitHub org that's behind SSO?

- Org UID `blt7f441a2870323433`, Project UID `blt20a497cfe39cfbd3`, Environment `development`.

_Keywords: connect GitHub org, SSO-protected GitHub, GitHub org permissions, approve Launch app installation, third-party app access policy, GitHub App approval, OAuth app restrictions, can't connect GitHub org, trial org._

ANSWER

## Root cause — GitHub org requires approving the Launch app

- The error was **not** a Launch fault — it was a **GitHub org permissions** issue. The customer's GitHub org requires their **team/admins to approve the Launch (GitHub) app for installation** before it can connect.
- This is standard GitHub behavior for orgs with **third-party application access restrictions / SSO enforcement**: an org admin must **authorize/approve the OAuth or GitHub App** before a member can install/connect it.

## What to do

1. Try connecting the **GitHub org again directly from the Launch UI** (rule out a transient error).
2. If it still errors, have a **GitHub org admin approve the Launch app installation** (GitHub → Org Settings → Third-party access / GitHub Apps → approve the pending request). For SSO-enforced orgs, ensure the app is also **authorized for SSO**.
3. Once approved, **complete the connection** in Launch.

## Workaround while waiting for internal approval

- Connect Launch to a **temporary GitHub org** to keep moving, then **re-point to the company org** once the admins approve the app.

## Generalized guidance (for future similar queries)

- **"Error connecting Launch to our GitHub org (SSO-protected)"** → almost always **GitHub-side org permissions**: the **Launch GitHub App must be approved by an org admin** (and SSO-authorized) before it can install. Not a Launch bug.
- **Triage:** retry from the Launch UI; if it persists, point the customer to their **GitHub org admin** to approve the app request.
- **Unblock now:** connect a **temporary org** and migrate to the real org after approval (re-pointing the repo connection is supported — see the org-migration entry).

## Status

- Identified as a **GitHub org permissions / app-approval** requirement. Customer awaiting their internal team's approval to allow the Launch app; using a **temporary GitHub org** as a workaround meanwhile. No fixed timeline for retrying with the company org.
