QUESTION

Customer's Launch project **lost its connection to the repository**. Clicking **"Repair Connection" does nothing**. Their org is **limited to one project** (already created), so they **can't create a new project** as a workaround to reconnect. The repo **was connected and working before**, then the connection **suddenly dropped** — GitHub App configuration looks fine on their end.

_Keywords: lost repository connection, repair connection does nothing, repair connection not working, GitHub App permissions, repo disconnected, one project limit, increase project limit, reconnect repository._

ANSWER

## First checks — GitHub App permissions

- The connection can drop if the **GitHub App lacks access to the required repo**. **Verify the GitHub App configuration** and that the **necessary permissions are granted** for that repository.
- Also work through the **Repair Connection troubleshooting** section in the Edge/Launch docs. (Note: the doc link doesn't always scroll to the troubleshooting section — go to it directly.)

## Workaround — increase the project limit to recreate/reconnect

- The customer's org is capped at **1 project**, so they can't spin up a new one to reconnect. As a workaround, **increase the project limit** so they can **create a new project and reconnect the repository**.

## If it persists (connected before, suddenly dropped, permissions fine)

- When the repo **was working and suddenly disconnected** with **GitHub App permissions intact** and **"Repair Connection" doing nothing**, it needs **deeper investigation** on the Launch side — collect the **org/project UID** and escalate, since the standard self-serve repair/recreate paths aren't resolving it.

## Generalized guidance (for future similar queries)

- **"Launch project lost repo connection / Repair Connection does nothing"** → (1) verify the **GitHub App has repo access/permissions**; (2) follow the **Repair Connection troubleshooting** doc; (3) if blocked by a **1-project org limit**, request a **project-limit increase** to recreate + reconnect; (4) if it was working and suddenly dropped with permissions fine, **escalate for investigation**.
- A **project-limit cap** can block the usual "create a new project to reconnect" workaround — raise the limit when that's the blocker.

## Status

- Recommended verifying **GitHub App permissions** + the **Repair Connection troubleshooting** steps, with a **project-limit increase** as the recreate-and-reconnect workaround. Customer confirmed permissions look fine and the repo was previously working → **needs further investigation** on the Launch side.
