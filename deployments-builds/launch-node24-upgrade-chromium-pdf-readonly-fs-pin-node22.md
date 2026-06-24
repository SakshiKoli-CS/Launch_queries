QUESTION

Customer **First Interstate Bank** (SF case **00057836**) hit production issues after a **platform-initiated Node.js runtime upgrade from v22.22.1 → v24.14.1**. Builds/deployments succeed, but **runtime** behavior changed:

1. **Chromium / PDF generation fails** (code **127**):
   `/tmp/chromium: error while loading shared libraries: libnspr4.so: cannot open shared object file` — a missing OS-level dependency (NSPR). Chromium appears platform-provided at `/tmp/chromium`.
2. **Read-only filesystem errors** when Next.js updates the prerender cache:
   `ENOENT: ... mkdir /var/task/sourcecode/.next/server/app/support`, `EROFS: read-only file system, open /var/task/.../login.html`.

They asked Launch to confirm Chromium/system-lib support on Node 24, whether `libnspr4` was removed, filesystem write expectations, a **rollback/pin to Node 22**, and official PDF guidance.

_Keywords: Node 24 upgrade, Node.js v24.14.1, pin Node 22, engines field package.json, Chromium libnspr4.so code 127, PDF generation failure, read-only filesystem, EROFS ENOENT .next prerender cache, NEXT_CACHE_DIR, /tmp writable, runtime regression._

ANSWER

## Rollback / pin to Node.js 22 (immediate option)

- Pin the Node version in **`package.json` → `engines`**:
  ```json
  { "engines": { "node": "22.x" } }
  ```
  This runs the app on the latest **22.x**. **If `engines` is not set, Launch defaults to Node 24.x.** Use this as the immediate rollback while the Chromium issue is investigated.
- Ref: Supported Node.js versions → setting the version in `package.json`.

## Read-only filesystem (NOT a Node 24 regression)

- The `ENOENT`/`EROFS` errors are **expected and not caused by the Node 24 upgrade** — Launch uses a **read-only filesystem across all Node versions** (Lambda-style). **Only `/tmp` is writable** at runtime; paths like `/var/task/sourcecode/.next/...` are read-only.
- Next.js (App Router) tries to write prerender/cache files under `.next`, which **isn't supported** on Launch. Redirect cache writes to a writable location:
  ```
  NEXT_CACHE_DIR = /tmp/.next/cache
  ```
- For caching/revalidation, use Launch's CDN-layer approach (see cache & revalidation guidance). Related: [launch-nextjs-cache-enoent-blank-pages-readonly-fs.md](launch-nextjs-cache-enoent-blank-pages-readonly-fs.md).

## Chromium / PDF generation

- The failure is a **missing system dependency (`libnspr4.so`)** when launching Chromium. **Temporary workaround: roll back to Node 22** (where it was working).
- Under **investigation on Node 24**; to help, share **how Chromium is used** — the library, how Chromium is bundled/invoked, whether it relies on a prebuilt binary, etc.
- **Important later finding:** on closer inspection, the **PDF paths returned intermittent 500s on BOTH Node v22 and v24** (alongside 200s). So the PDF failures are **not purely a Node-24 regression** and **likely originate application-side**. Recommendation: **add detailed logging around the PDF paths** and review server-side logs to find the root cause.

## Generalized guidance (for future similar queries)

- **"Pin/rollback the Node version on Launch"** → set **`engines.node`** in `package.json` (e.g. `"22.x"`); default is the latest major (Node 24.x) when unset.
- **`EROFS`/`ENOENT` on `.next` or any `/var/task/...` write** → **expected read-only FS on all Node versions** (only `/tmp` writable). Redirect Next.js cache with **`NEXT_CACHE_DIR=/tmp/.next/cache`**; don't attempt writes outside `/tmp`. (Same root as the `.next/cache` ENOENT/`cacheHandlers` entry.)
- **Chromium "libnspr4.so / code 127"** → missing system lib in the runtime image; rollback to a known-good Node version as a stopgap and share the Chromium setup for investigation.
- **Don't assume a Node upgrade is the culprit** — check whether the symptom also occurs on the prior version (here, PDF 500s happened on v22 too → app-side). Compare error rates across versions before calling it a platform regression.

## Status

- Provided Node-22 pinning, the read-only-FS clarification + `NEXT_CACHE_DIR`. Read-only FS confirmed **not** a Node 24 change. PDF/Chromium: rollback as stopgap; later evidence (500s on v22 and v24) points to **app-side** — customer asked to add logging and share findings. Investigation ongoing.
