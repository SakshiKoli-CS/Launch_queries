QUESTION

Customer from **MicroStrategy** (SF case **00053163**) has a **Next.js (15.5.9) App Router API route** trying to use **`after()`** or **`event.waitUntil()`** for **asynchronous background processing**. It **works locally but not on Launch**. They note both are tricky in a **serverless environment** depending on how aggressively the platform **kills the process once the response is sent**. Is there a way to make either work on Launch? Their fallback is a **separate API route fetched with `keepalive: true`**, but they'd prefer `after()` as the modern approach.

_Keywords: Next.js after() waitUntil() background processing, App Router API route, works locally not on Launch, serverless process killed after response, keepalive fetch workaround, Launch edge functions waitUntil supported, async background task._

ANSWER

## Background work after the response — serverless kills the process

- In a **serverless runtime**, the process can be **terminated as soon as the response is sent**, so work scheduled via **`after()` / `waitUntil()`** in a Next.js **API route** may not run reliably on Launch (even though it works locally, where the process persists).

## Options

1. **Launch Edge Functions support `waitUntil()`.** Moving the background work into a **Launch Edge Function** is a supported path for "do work after responding." Docs: [Launch Edge Functions](https://www.contentstack.com/docs/developers/launch/edge-functions).
2. **Customer's own workaround (valid):** trigger a **separate API route** for the background job and call it with **`fetch(..., { keepalive: true })`** so the request survives after the main response. (This is what the customer used to make it work.)

## Generalized guidance (for future similar queries)

- **"`after()` / `waitUntil()` works locally but not on Launch"** → expected for **serverless**: the process may be **reaped right after the response**, so post-response background work in an **API route** isn't guaranteed. Don't rely on it for must-run work.
- **Reliable patterns on Launch:** (a) **Launch Edge Functions, which support `waitUntil()`**; or (b) **offload to a separate endpoint** invoked with **`keepalive: true`** (or a queue/external worker) so the task isn't tied to the responding process's lifecycle.
- Related (long-running / async edge work): [edge function timeout — long-running LLM async pattern](launch-edge-function-timeout-long-running-llm-async-pattern.md).

## Status

- **Resolved.** Customer **made it work** — most likely via their described **`keepalive: true` separate-route** workaround. Also valid: use **Launch Edge Functions**, which **support `waitUntil()`**, for post-response background processing.
