QUESTION

Customer **Assurant / Concord** (Org `blt4781beb962759a43`, Project `67b744cb0424db6a7fd21eba`, SF case **00050411**) reports **Edge Functions fail to deploy in Development** (`67b744cc0424db6a7fd21ec1`) while the **same commit deploys fine to Staging** (`68dc1cf3981294d7f95a59ce`). Failure occurs **specifically during Edge Function deployment**, with **no detailed logs in the deployment UI**. Their `functions/[proxy].edge.js` handles regex/static-path redirects.

Ask: is there an **environment-level config/permission difference** between Staging and Development blocking the edge function deploy?

_Keywords: edge function fails to deploy one environment, same commit works staging fails development, no detailed logs in UI, environment variable difference, env var double quotes enclosing, extra spaces escaped \n, private key Form Mode, per-environment env var config, [proxy].edge.js redirects._

ANSWER

## Cause — environment variable difference (malformed env vars in the failing env)

- The failure was caused by a **difference in environment configuration — specifically the environment variables** in **Development** (not the code, since the **same commit deploys to Staging**).
- **Confirmed root cause:** two env vars in Development had been added by a developer **wrapped in double quotes (`"`)**. **Removing the quotes fixed the deploy.**

## What to check (env-var formatting)

- **Enclosing quotes** around keys/values — ❌ `key="1234"` → ✅ `key=1234`
- **Extra spaces.**
- **Escaped newline characters (`\n`)** / unexpected formatting.
- **Private keys:** add in the **correct multi-line format with REAL newlines, no enclosing quotes**, and **enter it in Form Mode** for it to be set properly:
  ```
  ✅ -----BEGIN OPENSSH PRIVATE KEY-----
     FAKEKEY...
     -----END OPENSSH PRIVATE KEY-----
  ❌ PRIVATE_KEY='-----BEGIN…\nFAKEKEY…\n-----END…-----'   (escaped \n + quotes)
  ```

## Generalized guidance (for future similar queries)

- **"Edge Function (or deploy) fails in ONE environment but the SAME commit works in another"** → it's an **environment-level difference, almost always the env vars**, not code or a permission/config gap. **Compare env vars between the two environments.**
- **Top culprit: malformed env vars** — **enclosing quotes** (the actual cause here), extra spaces, escaped `\n`. **Private keys** need real newlines, no quotes, set via **Form Mode**.
- The generic edge-deploy failure with **no detailed UI logs** is a known thin spot — audit env vars first rather than chasing the edge code.
- Related: [deploy fail malformed env vars (quotes/spaces/\n)](../deployments-builds/launch-deploy-fail-malformed-env-vars-quotes-spaces-newlines-stuck-deploying-18min.md), [env var unmatched quotes parsing](../deployments-builds/launch-deployment-failed-env-var-unmatched-quotes-parsing.md), [intermittent deploy fail — invalid env var](../deployments-builds/launch-intermittent-deploy-failure-invalid-env-var-config-poor-logs.md).

## Status

- **Resolved.** Cause: **malformed env vars in Development** — **two variables wrapped in double quotes** (added by a developer); the **same commit worked in Staging** because its env vars were clean. **Removing the quotes fixed it.** Not a permission/environment-config gap — purely env-var formatting.
