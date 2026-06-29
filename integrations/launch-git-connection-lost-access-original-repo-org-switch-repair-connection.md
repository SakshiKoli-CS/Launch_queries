QUESTION

Customer (Lee, demo env) sees a **Git connection error message** on a Launch project and wants the **quickest/simplest fix** (was about to just create a new Launch project instead).

_Keywords: Git connection error, GitHub connection no longer has access, switched GitHub org, disconnected OAuth connected different org, Repair Connection, demo environment, reconnect repo, original repo org access._

ANSWER

## Cause — the GitHub connection lost access to the original repo/org

- This message appears when the **GitHub connection used during project creation no longer has access** to the **original repository or organization.**
- Common trigger: you **created the project using GitHub Org A**, then **disconnected that OAuth connection and connected GitHub Org B** — so the connection backing the project no longer reaches the original repo.

## Quickest fix

1. **Disconnect and reconnect GitHub** with access to the **repo/org you originally used** (i.e. the one the project was created with).
2. Click **Repair Connection.**

This **restores access and fixes the Git connection immediately** — no need to recreate the project.

## Generalized guidance (for future similar queries)

- **Git connection error after switching GitHub orgs/accounts** → the project's connection is tied to the **org/repo it was created with**; connecting a *different* org breaks it. **Reconnect the original org/repo, then Repair Connection** — faster than recreating the project.
- Don't recreate the Launch project as a first resort — **Repair Connection** with the correct (original) GitHub access usually fixes it instantly.
- Distinguish from cases where **Repair Connection does nothing** despite correct permissions (deeper issue / project-limit blocker) — see [lost repo connection, repair does nothing](launch-lost-repo-connection-repair-does-nothing-project-limit.md) — and the platform-wide **OAuth/auth fault** case ([Bitbucket webhook 401](launch-bitbucket-webhook-401-unauthorized-deploy-stopped-repair-connection.md)).

## Status

- **Resolved.** Cause: GitHub connection **lost access to the original repo/org** (likely an org switch after project creation). Fix: **reconnect GitHub with access to the original repo/org → Repair Connection.** Customer confirmed fixed.
