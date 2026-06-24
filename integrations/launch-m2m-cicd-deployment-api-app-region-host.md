QUESTION

A prospect is setting up **automated deployments to Launch from Azure DevOps / Azure Pipelines** CI/CD. Source is in **Azure Repos**, so they use the **file-upload** method. The **`csdx launch` CLI only supports login-based auth (username/password)** — unsuitable for CI/CD (they can't hardcode an individual's credentials). What are the **machine-to-machine (M2M)** options? (Several follow-on errors arose while wiring up M2M.)

_Keywords: M2M deployment, machine-to-machine auth, Azure DevOps CI/CD Launch, csdx launch CLI login only, Launch Public API deploy, client_credentials invalid grant_type, M2M app vs standard app, developerhub_m2m_app plan key, launch:manage scope, access token invalid 105, eu-launch-api region host, token expiry 60 min._

ANSWER

## Use the Launch Public API with M2M OAuth (not the CLI)

- The **`csdx launch` CLI only supports login (username/password)** auth — **M2M support for the CLI is planned (~Q2)**. For now, **don't use the CLI for CI/CD**; call the **Launch Public API** directly with an **M2M OAuth token**.
- Reference: Launch API docs; example pipeline: `github.com/contentstack-launch-examples/launch-api-gitlab-ci-example`.

## Gotchas hit while setting up M2M (each with its fix)

1. **`"invalid grant_type"` (400) with `grant_type=client_credentials`** → you created a **standard app**, not a **Machine-to-Machine (M2M) app**. M2M auth (`client_credentials`) only works with an **M2M app** in Developer Hub.
2. **No option to create an M2M app** → the M2M app type is gated behind a **plan key**: enable **`developerhub_m2m_app`** on the org's plan. Once enabled, the **Create M2M app** option appears. (M2M is marked beta.)
3. **Scopes** → **`launch:manage`** is sufficient for **all** Launch operations (simpler than juggling `launch.projects.read` / `launch.projects.write`).
4. **Token expiry** → the M2M **access token expires in 60 minutes**; generate/use it within the window.
5. **`"access token invalid or expired or revoked"` (error 105)** even with a fresh valid token → **wrong API region host.** Use the **region-specific Launch API host**:
   - AWS **EU** → **`https://eu-launch-api.contentstack.com`** (not the default `https://launch-api.contentstack.com`).
   - Calling the wrong-region host makes a valid token appear invalid. **This was the actual fix.**

## Generalized guidance (for future similar queries)

- **"CI/CD deploy to Launch without user credentials"** → **Launch Public API + M2M OAuth** (`client_credentials`); the **CLI is login-only** (M2M CLI ~Q2).
- **`invalid grant_type` on client_credentials** → must be a **M2M app**, not a standard app.
- **Can't create an M2M app** → enable plan key **`developerhub_m2m_app`**.
- **Use `launch:manage` scope**; tokens last **60 min**.
- **`error 105 invalid/expired/revoked` with a fresh token** → almost always the **wrong region host**; use the region-prefixed API (`eu-launch-api…`, etc.). This is the classic "token works but is rejected" cause.

## Status

- **Resolved** — switched from CLI to **Launch Public API + M2M app** (plan key `developerhub_m2m_app` enabled, `launch:manage` scope); the final blocker was calling the **AWS-EU region host** `eu-launch-api.contentstack.com`. (Bearer token in the repro curl `<redacted>`; Project `69dcef3f0e7a6106816e1dfd`.)
