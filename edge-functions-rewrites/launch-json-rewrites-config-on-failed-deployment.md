QUESTION

Customer **Remote.com** (Launch project, EU region) raised two questions:

1. **Build-time page visibility:** "How can I be sure about the full list of pages that have been generated at build time and that will load instantly when visited? Is there a way to see it per deployment?"
2. **`launch.json` config lost on failed deployment (primary concern):** They generate a **`launch.json`** at build time to configure **critical rewrites and other deployment configs**. They observed that **when a deployment fails, these rewrites/configs appear to be lost / no longer applied**. With **Vercel**, only a **successful** deployment changed config — a failed deploy left the previous config in place. They expected the same on Launch and were worried Launch wipes config on failure.

_Keywords: launch.json, edge rewrites, URL rewrites, redirects, deployment failed, config lost, build-time generated config, cache priming, pre-rendered pages, instant load, Vercel migration, file at root level._

ANSWER

## Q1 — Seeing pages generated at build time / instant load

- **Key distinction: "generated at build time" ≠ "cached / will load instantly."** A page being pre-rendered (SSG) at build does **not** by itself guarantee it is warm in the edge cache and served instantly on first visit.
- To make pre-rendered pages load instantly, use **cache priming** (pre-warming the cache after deploy) rather than relying on build output alone.
- There is no per-deployment "instant-load page list" that is meaningful on its own — instant load depends on cache state, which is what cache priming controls.
- _(Customer self-resolved this question before the team responded — Q2 became the real issue.)_

## Q2 — `launch.json` rewrites "lost" after a failed deployment

**This is expected and safe behavior — config is NOT wiped on failure.**

- If a deployment **fails**, changes to `launch.json` are **not applied**. The **last successful deployment's config (rewrites, redirects, etc.) remains in effect.**
- This is **the same model the customer wanted from Vercel**: a failed deploy does **not** overwrite the live/previous config. The reassurance to give is: *your previous rewrites are still active; a failed build cannot replace them.*

**Why it looked "lost":**
- The customer **generates `launch.json` during the build step**. On a failed build, that file may not be produced/preserved for that attempt — but this only means the **new** config wasn't promoted. The **previously deployed config is untouched and still serving.**

**Requirements / fix:**
- Ensure `launch.json` is created at the **root level of the project** (not nested in a subdirectory or build output path), so Launch picks it up on a successful deploy.
- Confirm the build reliably emits `launch.json` at root before relying on build-time generation; if the build fails before writing it, the new config simply won't promote (old one stays).

**Reference doc:** Edge URL Rewrites — https://www.contentstack.com/docs/developers/launch/edge-url-rewrites

## Generalized guidance (for future similar queries)

- **"My rewrites/redirects/config disappeared after a deploy"** → First confirm whether that deploy **succeeded or failed**. On failure, Launch keeps the **last successful** config; nothing is lost. Config only changes on a **successful** deployment.
- **`launch.json` not taking effect** → Check it is at the **project root** and is present in the successful build's output.
- **"Pages should load instantly but don't"** → Distinguish **build-time pre-render** from **cache state**; use **cache priming** to warm the edge cache.
- **Migrating from Vercel** → Launch's failed-deploy config behavior matches Vercel's (failed deploy preserves prior config); set expectations accordingly.

## Status

- Guidance provided. Follow-ups sent to the account team; **no customer confirmation received** at last update (treated as likely resolved / awaiting closure).
