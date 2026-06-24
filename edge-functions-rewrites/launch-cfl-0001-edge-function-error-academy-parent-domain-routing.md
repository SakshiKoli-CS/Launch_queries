QUESTION

Users hit **Internal Server Error `CFL-0001`** when navigating to **Academy** (`https://www.contentstack.com/academy`) (SF case **00057140**, Org `bltb983c97aae052c5b`). The same error was also reported by some users on the **Dotcom** and **Docs** sites. The customer noted the **Academy codebase has no Edge Function** (it's a proxy from `.com`), so where is the error coming from?

_Keywords: CFL-0001, CFL0001, Internal Server Error edge function, edge function 500, academy error, no edge function in codebase, parent domain edge function, contentstack.com/academy routing, unhandled exception edge function, invalid response edge function._

ANSWER

## What CFL-0001 means

- **`CFL-0001`** occurs when the request **reaches the CDN layer but the Launch Edge Function fails to execute correctly**, returning a **500**.
- Typical causes: an **unhandled exception** in the Edge Function, or the function returning an **invalid/malformed response**. So it points to the **Edge Function implementation/code**.

## Key insight — the error is in the PARENT site's edge function, not Academy's

- The **Academy codebase has no Edge Function** — but Academy is served at **`https://www.contentstack.com/academy`**, so the request is **routed through the `contentstack-com` setup, where Edge Functions DO exist.**
- Therefore the failure is in the **`contentstack-com` Edge Function layer**, not the Academy code. (This also explains why **Dotcom and Docs** users saw it — same parent edge layer.)
- **General principle:** when a path is served under a **parent domain/app that proxies** (e.g. `/academy`, `/docs` under `www.contentstack.com`), the **parent's Edge Function applies** — debug the **parent** site's edge function, not the sub-app's codebase.

## Troubleshooting / fix direction

- **Check Launch Server Logs** for the Edge Function execution to find the exact failure point.
- In the (parent) Edge Function code:
  - Add **try/catch** around all critical sections to handle exceptions (avoid unhandled failures).
  - Add **detailed logging** to trace the execution flow.
  - Ensure the function **always returns a valid, properly structured response**.
- Docs: Troubleshooting CFL-0001.

## Generalized guidance (for future similar queries)

- **`CFL-0001` / Internal Server Error** → the **Edge Function failed** (unhandled exception or invalid response). Debug the edge function: server logs + try/catch + logging + guaranteed valid response.
- **"But our sub-app has no edge function"** → check whether it's **served under a parent domain/app** (e.g. `www.contentstack.com/academy`) whose **edge function the request flows through** — debug **that** parent edge function.
- **Same CFL-0001 across multiple sub-paths/sites** (academy, docs, dotcom) sharing a parent → reinforces that it's the **shared parent edge layer**.

## Status

- Identified the error as originating in the **`contentstack-com` Edge Function layer** (Academy has none of its own; it's proxied under `www.contentstack.com`). Recommended try/catch + logging + server-log review; tracked as **CL-3621**. Customer added the changes in pre-prod; awaiting logs to pinpoint the failure.
