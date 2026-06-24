QUESTION

Customer **Philips** (SF case **00057443**, EU, development environment) reported **deployments consistently failing** — across **multiple pipelines** and when triggered **directly via Launch**. Generic error:

> "Deployment failed: Please try to redeploy the site, or contact support for assistance."

Reproducible across attempts (not a one-off).

_Keywords: deployment failed, "Please try to redeploy the site", build failing, environment variable parsing error, unmatched quotes, special characters env var, ELOQUA_PASSWORD, syntax error env var, env var value rules._

ANSWER

## Cause — environment variable value with unmatched quotes (parsing error)

- The **build was failing due to a syntax/parsing issue while parsing an environment variable** — specifically, an env var value containing **unmatched single or double quotes (`'` or `"`)**. Here it was the **`ELOQUA_PASSWORD`** value.
- Unmatched/special quote characters in an env var value can break parsing during the build, surfacing as the **generic "Deployment failed… redeploy or contact support"** message (which by itself doesn't reveal the env-var cause).

## Fix

- **Check the env var value** (e.g. `ELOQUA_PASSWORD`) for **unmatched quotes** or stray special characters.
- **Quick workaround:** set a value **without special characters like quotes** and redeploy — the build succeeds.
- **Confirmed:** updating the value **resolved** the failing deployment.

## Notes / open follow-up

- There are **no documented guidelines yet** for env var values around special characters like quotes; documentation is being added.
- The **recommended safe way to use such characters when required** is to be confirmed (TBD with the team).
- The platform **validation isn't catching this earlier** (a clearer build-time error is being looked into) — which is why it shows only the generic failure message.

## Generalized guidance (for future similar queries)

- **Generic "Deployment failed: please redeploy or contact support" with no build error** → one common cause is a **bad environment variable value** — especially **unmatched quotes / special characters**. Audit recently-changed env vars first.
- **Quick test:** temporarily set the suspect var to a **plain value (no quotes/special chars)** and redeploy; if it builds, the original value was the culprit.
- **Reproducible across pipelines + direct deploys** points to **config (env var) or code**, not a transient platform incident.

## Status

- **Resolved** — the env var value (unmatched quotes) was the cause; updating it fixed the deployment. Docs/guidelines for env-var special characters and a clearer validation error are being worked on.
