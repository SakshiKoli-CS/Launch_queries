QUESTION

Does Launch have a **Management API**? Context: a team built an **internal deployment dashboard** that clones a stack and deploys via the **`csdx launch`** CLI. **Locally**, the CLI logs the **deployment URL**; but **on a hosted server the logs don't include the deployment URL**, which they need to surface to the user. Is there an **API call to fetch the deployment URL**?

_Keywords: Launch Management API, do we have management api, not public customer-facing, deployment URL from API, csdx launch CLI deployment url not in hosted logs, deployments API auth token, UI deployment details page API, fetch deployment details programmatically._

ANSWER

## Yes — Launch has Management APIs (currently internal, not public)

- Launch **does have Management APIs** — the **same ones the Launch UI uses** (e.g. on the **deployment details page**).
- They are **not public / not officially available to customers** yet; they **work with the UI or the CLI using the auth token.** *(A feature to expose Launch management APIs to customers is in play.)*

## Fetching the deployment URL via the deployments API

- You **can call the deployments API (the one the UI uses) to fetch the deployment URL and other deployment details** — this solves the "hosted server logs don't show the deployment URL" gap.
- Reference implementation calling the deployments API for URL + details: **`vishy1618/contentstack-launch-portfolio-builder` → `app/launch-repository.ts`** (the deployments call).

## Generalized guidance (for future similar queries)

- **"Is there a Launch Management API? / How do I get the deployment URL programmatically?"** → **Yes, Management APIs exist** (the **same APIs the Launch UI uses**), but they're **internal (not public), auth-token-based** for now; public exposure is on the roadmap.
- **To get a deployment URL when the `csdx launch` CLI doesn't log it (e.g. on a hosted/CI server)** → call the **deployments API** (as the UI's deployment-details page does); see the **portfolio-builder `launch-repository.ts`** example.
- For a **supported public path** to deployments, prefer **M2M OAuth + the Launch Public API** — see [M2M CI/CD deploy](launch-m2m-cicd-deployment-api-app-region-host.md) and [auth-token + org header](launch-api-authtoken-missing-organization-uid-header.md).

## Status

- **Answered.** Launch has **Management APIs** (same as the UI uses, e.g. deployment details), **not public** today (auth-token via UI/CLI; public exposure in play). For the **deployment-URL-not-in-hosted-logs** need, **call the deployments API** — referenced the `contentstack-launch-portfolio-builder` `launch-repository.ts` example.
