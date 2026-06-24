QUESTION

Customer **Crocs** (high priority) couldn't connect a new Launch project to their **Enterprise GitHub** account. Flow:
1. New Project → **Import from Git Repository** → **Connect Account** → GitHub authorization window.
2. They're **not a GitHub org admin**, so they click **Request Authorization**; the request shows **waiting for approval** in GitHub.
3. A **GitHub admin approves** the app.
4. On refresh, they get a **GitHub 404 page** — and now the **404 persists** on every retry, even after cancelling and restarting project creation.

Their broader concern: requiring a **GitHub org admin** to complete this is impractical — the person managing Launch projects usually **isn't** a GitHub admin. Is the only option to make a GitHub admin set up the Launch project (and deploy it) on their behalf? What's the real solution?

_Keywords: GitHub connect 404, GitHub 404 after admin approval, Connect Account 404, Launch GitHub App authorization, non-admin request authorization, enterprise GitHub, persistent 404, file upload import, CLI launch, no GitHub admin needed._

ANSWER

## What's happening

- The GitHub connection is a **user-initiated OAuth flow**: a user with the **required GitHub org permissions** authorizes the **Launch GitHub App** and completes the **installation**. If the **authorization/approval flow isn't completed correctly** (e.g. a stale session/state after the admin approves out-of-band), it can land on the **GitHub 404 page** — which can then stick on retries due to cached OAuth state.

## Unblock the GitHub connect

- **Retry the flow in Incognito / a fresh session** — this clears the stale OAuth state that causes the persistent 404, so the authorization + installation can complete cleanly.
- Ask the customer for a **screen recording of the exact steps** if it still breaks, to see where the flow stops.

## The real answer to "do we need a GitHub admin?" → No — use file upload

- **You do NOT need GitHub admin access to create/deploy a Launch project.** Instead of "Import from Git Repository," **import via file upload** — through the **Launch UI** or the **Contentstack CLI**. This path **does not require GitHub admin** approval or org permissions at all.
  - Import via file upload: https://www.contentstack.com/docs/developers/launch/import-project-using-file-upload
  - CLI for Launch: https://www.contentstack.com/docs/developers/cli/cli-for-launch
- This directly addresses the concern: a Launch user who isn't a GitHub admin can **set up and deploy the project themselves** via file upload, without routing everything through a GitHub admin.

## Generalized guidance (for future similar queries)

- **"GitHub 404 after the admin approved the Launch app / persistent 404 connecting GitHub"** → stale OAuth session/state; **retry in Incognito** to complete authorization + installation cleanly. Grab a screen recording if it persists.
- **"We can't/don't want to involve a GitHub org admin"** → **file upload (Launch UI or CLI) needs no GitHub admin** — recommend it when org policies restrict admin roles or the app-approval flow is blocked. (For the org-approval requirement itself, see [launch-connect-sso-protected-github-org-app-approval.md](launch-connect-sso-protected-github-org-app-approval.md).)
- GitHub Git-import requires the Launch GitHub App to be **authorized + installed by someone with org permissions**; if that's a blocker, **file upload is the no-admin path**.

## Status

- Recommended **Incognito retry** + **file-upload (UI/CLI)** as the no-GitHub-admin path; requested a screen recording for the 404 if it persists. Awaiting customer response.
