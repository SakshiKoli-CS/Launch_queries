QUESTION

Customer **Bibby Financial Services** (via iO Group NV; Stack `bltcaabb0e00358b2d2`, Project `682eeb26bd7f748ddf4e257b`, Env `683052c542f379b752e107c3`, SF case **00050534**) — production site on Launch shows an error to **end-users whenever there's a CDN cache MISS** (the request errors **before reaching their app**; **server logs show no related errors**, suggesting the server isn't reached). **Disabling browser cache loads the previously-failing page**, and once loaded it doesn't recur. They suspect the **recent scheduled-maintenance Redis cache update** corrupted/invalidated their cache, and plan a **redeploy to purge the CDN cache.**

Follow-up questions about Launch's behavior: is the restart mechanism **documented**? Where are **restart events logged** / can they be **automated/notified**? Are **multiple instances** running so there's **no downtime** during a restart?

_Keywords: CF001 on cache miss, error before reaching application, no server logs server not reached, Redis cache update maintenance suspected, redeploy purge CDN cache, CF001 500 application crashed unhandled runtime error, getServerEntryByUrl locale error, Launch queues requests background restart, multiple instances no downtime, server logs Starting message find crash, Log Targets, no restart automation/notification._

ANSWER

## Cause — CF001 = the application crashed (unhandled runtime error), surfaced on cache miss

- **CF001 is a 500** returned when a request reaches the Launch application but the **user application is in a broken/crashed state.** It occurs when the app **doesn't gracefully handle errors** that crash it.
- On a **cache HIT** the CDN serves the page (so it looks fine); on a **cache MISS** the request hits the (crashed) app → **CF001**. That's why **disabling browser cache "fixed" it** (forced fresh requests that eventually hit a healthy restarted instance) and why **their server logs looked empty** — the crash log is just **before** the restart.
- Launch reviewed the app logs and found errors **immediately preceding the CF001**, e.g.:
  ```
  Error in getServerEntryByUrl for content type: home_page  { url: '/', locale: 'sk-sk', ... }
  ```
  → it's an **application-code error** (unhandled), not a Redis/CDN corruption issue. **Customer should debug the app code** for the error block surfacing before CF001.

## How Launch handles a crashed app (the restart mechanism)

- When Launch detects the app is **broken/crashed**, it **queues incoming requests** (instead of immediately failing them), **restarts the app in the background**, and **sends the queued requests to the newly started process.**
- **Multiple instances run concurrently** — if one instance crashes (CF001), **other running instances keep serving** and traffic is routed to them while the crashed one restarts → **minimized downtime.**
- *(This behavior is in an upcoming documentation update; not fully documented yet at the time.)*

## How to find the crash cause in logs

- **Server Logs** section: look for **server start messages**, e.g.:
  ```
  ▲ Next.js 15.5.3
  ✓ Starting...
  ```
  The **log entry immediately BEFORE a "Starting…"** is the **crash that triggered the restart** (e.g. an unhandled error, max header size exceeded, etc.).
- For full retention/monitoring, configure **Log Targets** to stream logs externally.
- **Launch does NOT support configuring automation/notifications on restart events** currently.

## Generalized guidance (for future similar queries)

- **"Error only on cache MISS / disabling cache 'fixes' it / no server-log errors"** → the app is **crashing (CF001)**; cache HITs mask it, MISSes expose it. **Not a CDN/Redis corruption** by default — look at the **app's unhandled runtime errors.**
- **Find the cause:** in **Server Logs**, locate `✓ Starting...` (a restart) and read the **error immediately before it** — that's the crash. Use **Log Targets** for retention.
- **Restart behavior:** Launch **queues requests + restarts the crashed instance in the background**, and **other concurrent instances keep serving** → minimal downtime. **No restart-event automation/notifications** today; **test in dev** to catch crash-causing errors before prod.
- Fix is **graceful error handling in the app** so a single bad path (e.g. a failed `getServerEntryByUrl` for a locale) doesn't crash the process.
- Related: [CF001 resource saturation / memory leak](launch-cf001-server-failed-to-start-resource-saturation-memory-leak.md), [UVA CF001 site-slow CPU/mem](launch-uva-cf001-site-slow-cpu-memory-100-redeploys-masked-add-caching.md), [server logs 5k limit / Log Targets](launch-server-logs-ui-5000-entry-limit-use-log-targets.md).

## Status

- **Explained — application-side.** CF001 surfacing on **cache miss** = the **app crashing on an unhandled error** (`getServerEntryByUrl … locale 'sk-sk'`), **not** Redis/CDN corruption. Shared the crash logs for the customer to **debug their code**. Clarified Launch's **queue + background-restart + multi-instance** mechanism, how to **find restarts in Server Logs** (`✓ Starting...`), that **multiple instances avoid full downtime**, and that **restart automation/notifications aren't supported** (use **Log Targets** + dev testing).
