QUESTION

Customer **Sophos / MicroStrategy** (Org `blt7eaaa4a78adf9602`, Project `68be5c7df1773c96089c2859`, Env `6937c76b2dadd1f7c7e27825`, **aws-na**, `sophos-production.contentstackapps.com`, SF case **00053098**) sees **"failed to start server"** errors in Launch server logs, **sporadically since the first deployment** (random intervals, no clear pattern). Initial theory was the app hardcoded **port 4000** instead of `process.env.PORT` → **`EADDRINUSE`**, but the customer says they **didn't hardcode the port** and use the **default Next.js config** (port 3000 locally).

They ask: (1) official docs on **how Launch assigns/uses the `PORT` variable**? (2) is **PORT consistent per deployment or per request**? (3) **Next.js-specific guidance** for deploying on Launch?

_Keywords: failed to start server, EADDRINUSE port 4000, process.env.PORT, Launch PORT environment variable, PORT per deployment not per request, hardcoded port, Next.js default 3000, document is not defined SSR ReferenceError, ApolloClient Int cannot represent non-integer value, 500 errors specific pages, Next.js on Launch guide._

ANSWER

## How Launch uses PORT

- **Apps must listen on the `PORT` env var Launch provides at runtime** — Launch sets `PORT` and expects the app to bind to it so it can manage **routing and deployments**. **Don't hardcode a port** (e.g. 4000); hardcoding causes **`EADDRINUSE` / "failed to start server."**
- **No dedicated doc** explains the internal mechanics of how Launch assigns `PORT`; the usage rule is simply: **read `process.env.PORT` and listen on it.**
- **PORT is assigned per deployment instance, NOT per request.** It **may vary between deployments** but stays **consistent for a given instance** — this lets Launch run **multiple instances concurrently without conflicts.**
- **Next.js guidance:** use the official **Next.js on Launch guide** (supported routing models, rendering/caching behavior, deployment best practices).

## During investigation — other 500s found (separate app bugs, not PORT)

Repeated **500s** surfaced on specific pages, which are **application-side bugs** (worth fixing to stabilize the env, but distinct from the PORT issue):
- **`ReferenceError: document is not defined`** on blog category/tag pages (`/en-us/blog/category/...`, `/en-us/blog/tag/...`, etc.) → **server-side rendering touching browser-only globals** (`document`). Code that uses `document`/`window` must be **guarded for SSR** (run only client-side, e.g. in `useEffect` or with a `typeof document !== 'undefined'` check / dynamic import with `ssr: false`).
- **ApolloClient GraphQL `"Int cannot represent non-integer value"`** on `/it-it/integrations/integration-partners`, `/ja-jp/...` → a **GraphQL schema/data type mismatch** (a non-integer value sent where an `Int` is expected) in the app's queries/data.

## Resolution on the platform side

- The **"port 4000 error" was fixed from the Launch side**; customer asked to **redeploy** and confirm.

## Generalized guidance (for future similar queries)

- **"failed to start server" / `EADDRINUSE`** → the app must **bind to `process.env.PORT`** (Launch supplies it). **Never hardcode a port.** Even with default Next.js config, verify no custom server / script binds a fixed port.
- **PORT semantics:** **per deployment instance, not per request**; varies between deployments, constant within an instance (enables concurrent instances).
- **No internal-mechanics doc for PORT** — the contract is just "listen on `PORT`." For Next.js, follow the **Next.js on Launch guide**.
- **`document is not defined` / `window is not defined` at runtime = SSR executing browser-only code.** Guard with client-only execution (`useEffect`, `typeof window !== 'undefined'`, or `next/dynamic` `ssr:false`). This is an **app bug**, not a Launch issue.
- **Don't conflate** the platform PORT/start-server issue with **app-level 500s** (SSR `document`, GraphQL type errors) found during triage — fix both, but track separately.

## Status

- **PORT/"failed to start server": fixed platform-side** (port 4000 error) → redeploy to confirm. PORT is **per-deployment-instance, listen on `process.env.PORT`, don't hardcode**. Separately surfaced **app-side 500s** — **`document is not defined`** (SSR guarding needed) and **ApolloClient `Int` type mismatch** — recommended for the customer to fix to stabilize the environment.
