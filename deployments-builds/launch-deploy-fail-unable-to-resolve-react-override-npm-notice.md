QUESTION

Customer's **deployments started failing after they changed a few dependencies** in `package.json`. Build error during dependency install:

```
Unable to resolve reference $react
```

(Node 22.20.) Follow-up: after fixing, they noticed an npm log line and worried npm had been **downgraded a major version** during deployment:
- failing install log: `npm notice New major version of npm available! 10.9.7 -> 11.12.1`
- older (Mar 26) deploy log: `npm notice New minor version of npm available! 11.11.1 -> 11.12.0`

_Keywords: Unable to resolve reference $react, package.json overrides $react, dependency install fails after change, npm placeholder reference, npm downgrade, npm notice new version, deployment failing dependencies, overrides exact version._

ANSWER

## Cause — unresolved `$react` reference in package.json (not a Launch issue)

- The error **`Unable to resolve reference $react`** comes from the **updated `package.json`** — a **`$`-prefixed reference (`$react`) in `overrides`** that **wasn't resolved to an exact version**. This is a **known npm behavior** (publicly documented on GitHub), **not a Launch problem**.

## Fix — replace `$react` with the exact version in `overrides`

- Replace the `$`-reference with the **explicit version**, e.g.:
  ```json
  "overrides": {
    "eslint": "^9.39.3",
    "react": "^19.2.1"
  }
  ```
- **Verify the install works locally first** (`npm install`) before testing on Launch — this is a **package.json/npm config** issue, so it reproduces locally and saves a deploy cycle. Confirmed: a small deploy succeeded after the fix.

## The npm "downgrade" is a misread — it's just a version notice

- Lines like `npm notice New major version of npm available! 10.9.7 -> 11.12.1` are **informational "a newer version is available" notices** — they do **NOT** mean npm was **downgraded** (or upgraded) during the deployment.
- To confirm the **npm version actually used**, **review the complete deployment logs** (the notice text alone isn't evidence of a version change).

## Generalized guidance (for future similar queries)

- **`Unable to resolve reference $react` (or `$<pkg>`) on install** → a **`$`-prefixed override/reference in `package.json` not resolved to a concrete version**; replace it with the **exact version**. It's an **npm/package.json** issue, not Launch.
- **Reproduce locally first** for any dependency-install failure — if `npm install` fails locally with the same config, it's not a platform problem.
- **`npm notice New … version available`** in build logs is **just an availability notice**, not a version change/downgrade. Check full deploy logs for the actual npm version if in doubt.

## Status

- Identified the `$react` override as the cause (npm/package.json, not Launch); fixed by using an **exact version** → deploy succeeded. Clarified the **npm notice ≠ downgrade**; requested full logs to confirm the npm version used.
