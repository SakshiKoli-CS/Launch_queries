QUESTION

Customer **McKinsey** (prospect, PoC) has a Launch environment whose deployments **keep failing**. Observations:

- UI says **deployment failed**, but the **status stays stuck on "deploying"** so no other action can be taken — **no console or network errors.**
- It only flips to a **failed state after ~18 minutes.**
- The underlying issue turned out to be **missing / misconfigured environment variables.**

They ask what causes the failure and why it takes 18 minutes to report failure.

_Keywords: deployment failed stuck on deploying, 18 minutes to fail, no console network errors, missing env variables, misconfigured environment variables, enclosing quotes, extra spaces, escaped newline \n private key, must redeploy after changing env vars, slow failure status._

ANSWER

## Cause — misconfigured environment variables

Deployment failures (including the "missing env variables" the customer saw) commonly come from **malformed env vars** in the Launch environment. Check for:

- **Enclosing quotes** around keys/values — ❌ `key="1234"` → ✅ `key=1234`
- **Extra spaces** — ❌ `key=a bc` → ✅ `key=abc`
- **Escaped newline characters (`\n`)** / wrong multi-line formatting, especially for **private keys**.

**Private key — correct vs incorrect:**
```
✅ Correct (real newlines, no quotes):
-----BEGIN OPENSSH PRIVATE KEY-----
FAKEKEY1234567890abcdefghijklmnopqrstuv
FAKEKEY1234567890abcdefghijklmnopqrstuv
-----END OPENSSH PRIVATE KEY-----

❌ Incorrect (escaped \n + enclosing quotes):
PRIVATE_KEY='-----BEGIN OPENSSH PRIVATE KEY-----\nFAKEKEY...\n-----END OPENSSH PRIVATE KEY-----'
```

**You must trigger a new deployment after adding/modifying env vars.** Docs: Environment Variables in Launch.

## The "stuck on deploying for ~18 min then fails" behavior

- The status appearing **stuck on "deploying"** and only flipping to **failed after ~18 minutes** (with **no console/network errors**) suggests a **background step that isn't surfaced in the build logs** — or a **final step taking a very long time** for that app — before the failure is reported.
- Flagged as a **possible UX/logging improvement**: surface the real failure faster / in the logs instead of a long silent "deploying" state. (Not customer-fixable; the deploy *does* eventually fail correctly.)

## Generalized guidance (for future similar queries)

- **Deploy fails / "missing env variables"** → audit env vars for the three classic formatting mistakes: **enclosing quotes**, **extra/internal spaces**, and **escaped `\n`** (multi-line values like private keys must use **real newlines, no quotes**). **Redeploy after any env-var change.**
- **Status stuck on "deploying," no errors, fails only after many minutes** → likely an **un-surfaced background/final step**; the deploy is genuinely failing, just **reported slowly**. Don't read the long "deploying" state as a hang to be cancelled — wait for the failed state, then check env vars/logs.
- Related: [deploy fails on env var with unmatched quotes (parsing)](launch-deployment-failed-env-var-unmatched-quotes-parsing.md), [intermittent deploy fail — invalid env var](launch-intermittent-deploy-failure-invalid-env-var-config-poor-logs.md), [env var 100-key limit](launch-environment-variable-limit-100-consolidate-workaround.md).

## Status

- **Resolved.** Cause: **misconfigured environment variables** (quotes / spaces / escaped `\n`); fixing the formatting + redeploying resolved it. Separately flagged the **~18-min "deploying"-then-failed** delay as a **logging/UX improvement** (background/final step not surfaced in logs).
