QUESTION

Customer **Philips** (Org `bltb414e71bd2e61150`, Project `67a5d6832181bda09f767cb4`, Production, Next.js, SF case **00050169**) sees recurring **CF001** after their first release. Logs show their app's **`fetcher:` "bad response"** where a **`POST /api/c/search-hub`** returns **500** with:
```
Internal Server Error CF001. Server failed to start. Please check Launch Server Logs for more details.
```
They identified the **500 is thrown by their own application** on that endpoint (they plan to remove/refactor it). They ask: **is their app's 500 a trigger/contributing factor to the CF001**, and how is CF001 generated / what mitigation is there on Launch's side?

_Keywords: CF001 Server failed to start, app 500 /api/c/search-hub, fetcher bad response, CF001 right after requested entry doesn't exist error_code 141, Taxonomy API 400 empty $above $eq_above, application-side crash not Launch, correlate 5xx with CF001, Next.js._

ANSWER

## Yes — CF001 is triggered by the application crashing (correlated with app-side errors)

- **CF001 = the user application crashed / failed to start**; here it's **driven by the app's own errors**, not a Launch fault. Correlating logs with the 5xx occurrences:
  - **CF001 appears immediately AFTER the app error `The requested entry doesn't exist` (`error_code: 141`)** — i.e. an **unhandled failure fetching content** (e.g. `getSingleContent` / `getConsumerLocale` for a content type + locale) crashes the process → next request 500s with CF001.
  - The **app's own 500 on `POST /api/c/search-hub`** ("fetcher: bad response") is part of the same unhandled-error pattern.
- So the customer's **app-side 500 IS the trigger/contributing factor** — Launch surfaces CF001 when the app is in a broken state.

## Contributing cause — Taxonomy API 400s from empty query params

- The app makes **Taxonomy API calls with EMPTY values for the `$above` and `$eq_above` query parameters** → the Taxonomy API returns **400**. These failed calls feed the crash/error path.
- **Fix:** **provide valid values for `$above` / `$eq_above`** (don't send them empty). If populating them doesn't resolve it, it's escalated to the **Taxonomy team.**

## Mitigation

- Root fix is **app-side: handle these errors gracefully** (missing entry / bad-response / Taxonomy 400) so a failed fetch doesn't crash the server → no CF001. The customer's plan to **remove/refactor the `/api/c/search-hub` endpoint** addresses the immediate trigger.
- Launch also **queues requests and background-restarts** a crashed instance, and **other instances keep serving** — but the durable fix is preventing the unhandled error.

## Generalized guidance (for future similar queries)

- **CF001 recurring after release + app logs show `fetcher: bad response` / a 500 from an app endpoint** → the **app is crashing on an unhandled error**; CF001 is the symptom, not the cause. **Correlate CF001 timestamps with the app error immediately preceding** — here it was **`The requested entry doesn't exist` (error_code 141)**.
- **Watch for Taxonomy API 400s from empty query params** (`$above`, `$eq_above` sent empty) — populate them with valid values; empty params are a common self-inflicted 400 that can cascade into a crash.
- Fix is **graceful error handling** for missing-entry / bad-response / dependency-400 paths so one bad fetch doesn't take down the server.
- Related: [CF001 on cache miss → app crash / restart mechanism](launch-cf001-on-cache-miss-app-crash-unhandled-error-restart-mechanism.md), [CF001 resource saturation](launch-cf001-server-failed-to-start-resource-saturation-memory-leak.md), [find crash in Server Logs before `✓ Starting...`](launch-cf001-on-cache-miss-app-crash-unhandled-error-restart-mechanism.md).

## Status

- **Explained — application-side.** CF001 is **triggered by the app crashing on unhandled errors**, correlating **right after `The requested entry doesn't exist` (error_code 141)** and the **`/api/c/search-hub` 500** ("fetcher: bad response"). Contributing: **Taxonomy calls with empty `$above`/`$eq_above` → 400** (fix: valid values; else Taxonomy team escalation). Not a Launch fault; customer refactoring the endpoint + should add **graceful error handling**.
