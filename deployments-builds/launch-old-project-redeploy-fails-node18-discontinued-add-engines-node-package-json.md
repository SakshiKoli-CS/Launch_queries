QUESTION

Customer has an **old Launch project** (built a couple of months ago; local dev was on Node **18.x** then). **Redeploying it today fails** with a build error. But **creating a NEW project from the same repo deploys fine.** Why does the original project fail while a fresh one with identical code works?

_Keywords: old project redeploy fails, new project same repo works, Node 18 discontinued end of life, default node version changed 18 to 22, engines field package.json node 22.x, deployment defaulted to Node 18 previously, pin node version, unsupported node version build error, redeploy after node deprecation._

ANSWER

## Cause — the old project is pinned to the now-discontinued Node 18

- **Node 18 has been discontinued** on Launch. 
- **Historically, if a project didn't define a Node version, deployments defaulted to Node 18.** The **old project was created under that default**, so it's effectively **tied to Node 18** — which **no longer builds** now that 18 is removed.
- **New projects now default to Node 22.** That's why the **fresh project from the same repo deploys fine** — it picks up the new **Node 22** default, while the **old one is still trying to use the retired Node 18.**

## Fix — pin the Node version explicitly via `engines` in `package.json`

```json
"engines": {
  "node": "22.x"
}
```
- Add this (with the Node version you need), **commit, and redeploy** → the old project builds on the specified version.
- Defining `engines.node` explicitly is the **robust practice** — it removes dependence on whatever the platform default happens to be, so a **future default change won't silently break redeploys.**

## Generalized guidance (for future similar queries)

- **"Old project won't redeploy but a new project from the same repo does"** → almost always a **runtime/default drift**: the old project is **pinned to (or defaulted to) a now-removed Node version** (e.g. **Node 18**), while new projects default to the **current version (Node 22)**.
- **Fix: set `engines.node` in `package.json`** to a supported version (e.g. `"22.x"`), commit, redeploy.
- **Always pin `engines.node`** so deployments don't rely on the implicit default — this is what makes builds reproducible across time and prevents breakage when Launch retires an old Node version.
- Related: [Node 24 upgrade broke Chromium/PDF + read-only FS → pin Node 22](launch-node24-upgrade-chromium-pdf-readonly-fs-pin-node22.md), [build JS heap OOM → `NODE_OPTIONS=--max-old-space-size`](launch-build-js-heap-out-of-memory-node-options-max-old-space-size.md), [deployment stuck / hangs](launch-deployment-stuck.md).

## Status

- **Resolved.** Old project failed because it was on the **discontinued Node 18** (the old no-`engines` default); **new projects default to Node 22** (hence the fresh project worked). Fix: **add `engines: { node: "22.x" }` to `package.json`**, commit, and redeploy.
