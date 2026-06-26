QUESTION

Customer (**NBCU**, SF case **00052691**; same error also hit **Milestone Systems**, Project `68d29b7ceaa2ca1bb35ac4d0`, Azure EU) — Launch deployment **fails during dependency install at `npm error command sh -c sst install`**. Redeploying does nothing.

- Milestone detail: **same code worked a month ago**; broke **only after changing an env variable and redeploying** (right before a go-live).
- With `--print-logs`, the real error surfaced:
  ```
  npm error command sh -c sst install --print-logs
  npm error time=... level=ERROR msg="exited with error" err="user: Current requires cgo or $USER set in environment"
  Installing dependencies failed
  Deployment failed: Please try to redeploy the site, or contact support for assistance.
  ```
- `npm install` / `sst install` **work locally.** Repo is a **monorepo** where other projects use **SST**, so SST is being used to trigger the frontend build too (not strictly required for the frontend).

_Keywords: sst install Launch build fail, npm error command sh -c sst install, user Current requires cgo or $USER set in environment, USER=root env variable fix, --print-logs, monorepo SST, avoid sst install in build phase, deployment failed redeploy does nothing, pulumi binary, env var change triggered failure._

ANSWER

## Immediate fix — set `USER=root`

- The SST error **`user: Current requires cgo or $USER set in environment`** means the build process had **no `$USER`** set. **Setting `USER=root` in the Launch Environment Variables resolved it** (SST/its Go-based tooling needs `$USER` present).
- Surfacing it required re-running with **`--print-logs`** (`sst install --print-logs`) — the generic "redeploy or contact support" message hides the real cause.

## Recommendation — don't run `sst install` in the Launch build phase

- **SST does not perform any deployment or infrastructure changes on Launch**, so **avoid using `sst install` as part of the Launch build phase.** It's being pulled in only because the **monorepo** uses SST for other projects — but for the **Launch frontend build it's unnecessary** and a source of failures (SST behavior changed recently; also seen picking up **global pulumi binaries** from the build image instead of npm-installed ones).
- Decoupling the Launch build from `sst install` avoids this whole class of issues going forward.

## Note on "it worked a month ago, broke after an env-var change"

- The redeploy (triggered by the env-var change) **re-ran the install with current tooling**, exposing the **`$USER`/SST** issue that the older cached/previous deploy hadn't hit. The **env var itself wasn't the cause** — redeploying surfaced an **SST/build-image change**. (General pattern: a previously-good project failing right after a redeploy can reflect **upstream tooling/image drift**, not the change you made.)

## Generalized guidance (for future similar queries)

- **`sst install` fails in the Launch build with `user: Current requires cgo or $USER set in environment`** → **set `USER=root`** in env vars. (SST's Go tooling needs `$USER`.)
- **Use `--print-logs`** to surface the real SST error behind the generic "Deployment failed: redeploy or contact support."
- **Don't run `sst install` in the Launch build phase** — SST does no deploy/infra work on Launch; if it's only present because of a **monorepo**, decouple the frontend build from SST.
- **"Worked a month ago, broke after a small change + redeploy"** → suspect **build-image/tooling drift** (SST, pulumi binaries) exposed by the redeploy, not necessarily your change.
- Related: [electron socket-hang-up / large-binary install](launch-uva-frequent-deploy-failures-ssg-rebuilds-network-electron-ssr-recommendation.md), [intermittent deploy fail — invalid env var](launch-intermittent-deploy-failure-invalid-env-var-config-poor-logs.md).

## Status

- **Resolved.** Cause: SST install failing because **`$USER` wasn't set** (`requires cgo or $USER`); **`USER=root` env var fixed it.** Recommended **not using `sst install` in the Launch build phase** (SST does no Launch deploy/infra work; monorepo convenience only). NBCU's UPAH build was failing for an **unrelated** reason (under their own investigation). Case closed after follow-ups.
