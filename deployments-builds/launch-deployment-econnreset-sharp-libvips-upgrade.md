QUESTION

Customer **MicroStrategy / Strategy** (SF cases **00057329 / 00057832**) reported **intermittent deployment failures** while **installing a package**, with **`ECONNRESET`** errors during the build. Reproducible intermittently across deployments (e.g. failed deployment **#1454**, Strategy Metrics and IR project, Production env). A deploy also once showed a transient **502 Bad Gateway (cloudflare)** that succeeded on retry.

_Keywords: deployment failed installing package, ECONNRESET, sharp postinstall, libvips download GitHub, sharp version 0.33, intermittent build failure, TCP reset, 502 Bad Gateway deploy retry, npm install network error._

ANSWER

## Root cause — `sharp` postinstall downloading libvips over a long HTTPS connection

- The **`ECONNRESET`** errors occur in the **`sharp` postinstall** step. In **versions prior to 0.33**, `sharp` **downloads libvips from GitHub Releases over a long-lived HTTPS connection**, which is **susceptible to TCP resets from intermediate network hops** — causing **intermittent** install/deploy failures.

## Fix — upgrade `sharp` to ≥ 0.33

- Check the **`sharp` version in `package.json`**; if it's **below 0.33**, upgrade to **`"sharp": "^0.33.5"`**.
- From **0.33 onward, `sharp` no longer performs the GitHub download** — the binaries are distributed as **npm optional dependencies via the npm registry**, eliminating this class of failure. **0.33.5 is compatible with Next.js 14.**
- Reference sharp issues: GitHub `lovell/sharp` #1850, #2957, #3316, #3750 (the 0.33 installation change).

## Note on the transient 502

- A one-off **502 Bad Gateway (cloudflare)** during a deploy that **succeeded on retry** is a separate **transient** blip — not the `sharp`/`ECONNRESET` root cause; retrying the deploy is the action there.

## Generalized guidance (for future similar queries)

- **"Intermittent deployment/build failures with `ECONNRESET` while installing packages"** → check for **`sharp` < 0.33**, which **downloads libvips from GitHub** over a connection prone to TCP resets. **Upgrade to `sharp@^0.33.5`** (binaries come via npm optional deps, no download).
- More broadly, **`ECONNRESET` in a postinstall** points to a dependency **fetching a binary from an external host during install** — pin to a version that ships prebuilt binaries via the npm registry instead.
- **Transient 502 (cloudflare) on a single deploy that retries clean** → treat as a one-off; retry.

## Status

- Recommended **upgrading `sharp` to `^0.33.5`** (from a pre-0.33 version that downloads libvips). Customer deploying the change and monitoring. (Related, same customer, separate root cause — login 524 timeout from a chunked redirect: [launch-524-520-login-redirect-chunked-no-terminator-content-length.md](../networking-connectivity/launch-524-520-login-redirect-chunked-no-terminator-content-length.md).)
