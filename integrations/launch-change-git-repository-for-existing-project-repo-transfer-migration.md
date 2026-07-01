QUESTION

Customer **Arke / The Jackson Laboratory** (Org `blt4b5cdc8803089fa4`, Azure NA, SF case **00050193**) is **transferring the GitHub repo** for the Jax.org project from `Arke-Systems/jax-frontend` to a new repo under a new org (`jax-org/…`). They want to **update the linked Git repository for an existing Launch project — WITHOUT deleting and re-creating the project.** Is that possible?

_Keywords: change GitHub repository for existing Launch project, switch linked git repo, repo transfer to new org, update git connection without recreating project, migration repository mapping, Repair GitHub Connection, install GitHub app new org, change-git-repository docs._

ANSWER

## Yes — you can change the linked Git repo without recreating the project

- **Switching the GitHub repo for an existing Launch project is supported** (no need to delete/recreate the configuration).
- **There is now a self-serve documented process:** **https://www.contentstack.com/docs/launch/change-git-repository-for-a-project** — share this doc for current requests.

## Process (as performed here — manual migration, pre-docs)

At the time this case ran, it required a **support-assisted migration**:

1. **Transfer the repo to the new org first**, then **inform Launch** (support runs a **migration to update the project's repository mapping**). Details needed:
   - **Current** GitHub repo + org name + URL
   - **New** GitHub repo + org name + URL
2. After the migration, the customer:
   - **Establishes a new Git connection in the Launch UI**, then
   - **Project Settings → Repair GitHub Connection** to complete the setup.
3. **On the call you need someone with access to install the GitHub App in the NEW org**, and ideally the **Launch project owner** + **admin on the new repo**.
- A **30–45 min call** was used to run the migration and validate together.

## Generalized guidance (for future similar queries)

- **"Change the GitHub repo linked to an existing Launch project (e.g. repo transferred to a new org) without recreating it?"** → **Yes.** **Point them to the docs: [Change Git Repository for a Project](https://www.contentstack.com/docs/launch/change-git-repository-for-a-project).**
- Key prerequisites regardless of method: **transfer the repo to the new org first**; have someone who can **install the GitHub App in the new org**; then **new Git connection + Repair GitHub Connection** in the Launch UI. The **project owner + new-repo admin** should be involved.
- (Historically this was a **support-run migration**; it's now self-serve per the doc.)

## Status

- **Resolved.** Changing the linked repo **is supported without recreating the project.** Handled via a **support-assisted migration** (repo mapping update) + **new Git connection + Repair GitHub Connection** (with GitHub-App install access in the new org). **Now documented** — use **https://www.contentstack.com/docs/launch/change-git-repository-for-a-project** for future requests.
