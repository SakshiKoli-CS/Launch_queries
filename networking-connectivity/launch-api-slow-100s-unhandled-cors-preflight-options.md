QUESTION

Customer's **commerce APIs (Next.js)** on Launch (Red Panda Resort site, Project `691c97907ecdc6639dd4c3e6`, AWS) suddenly responded in **50–100 seconds** (vs **~3–4s locally**), starting "today," **only on the hosted environment**. A basic PDP took **>100s** to load, impacting all demos. They asked if there was a Launch issue/change.

_Keywords: API slow 50-100 seconds, 100s response time, PDP slow, CORS preflight OPTIONS slow, OPTIONS not handled, preflight 101s TTFB, fetch instant waiting on preflight, PATCH slow, unhandled HTTP verbs, works locally slow on Launch, cross-origin._

ANSWER

## Root cause — unhandled CORS preflight `OPTIONS` request

- The slow request is the **CORS preflight `OPTIONS`** call, **not** the actual data fetch. Measured on the OPTIONS request:
  ```
  DNS Lookup: 0.02s · TCP Connect: 0.03s · TLS: 0.05s · Time to First Byte: 101.03s · Total: 101.04s · 200
  ```
  DNS/TCP/TLS are instant — only **TTFB is ~101s**, i.e. the **server (app) takes ~100s to respond to the preflight**.
- Running the **actual GET fetch alone is instant** — the browser is simply **waiting for the preflight to finish** before sending it, so the whole request appears to take 100s.
- The app **doesn't handle the `OPTIONS` preflight** (and similarly **`PATCH`** and other verbs **not explicitly handled** were slow). Unhandled methods hang ~100s.
- **No Launch network issue** — DNS/TCP/TLS are fast; this is the **application not handling OPTIONS/preflight correctly.** Works locally because there's **no cross-origin preflight** locally.

## Fix — handle OPTIONS (and other verbs) explicitly in the app

- Add an **`OPTIONS` (preflight) handler** to the API routes, returning the appropriate CORS headers quickly. Also explicitly handle other methods used (e.g. **`PATCH`**) so they don't hang.
- **Confirmed:** adding the preflight/OPTIONS handling **resolved** the 100s latency.

## Generalized guidance (for future similar queries)

- **"API takes ~50–100s on Launch but is instant locally"** → inspect the **`OPTIONS` preflight** timing, not just the fetch. If the OPTIONS request's **TTFB is ~100s** while DNS/TCP/TLS are instant, the **app isn't handling preflight** — the browser blocks the real request behind it.
- **Cross-origin only** (browser sends preflight for cross-site requests with auth/custom headers like `Authorization`, `x-store-token`) → explains "works locally, slow when hosted/embedded on another domain."
- **Fix in the app:** add an **OPTIONS handler** + explicitly handle every HTTP verb the API uses (GET/POST/**PATCH**/DELETE). Unhandled methods can hang ~100s. Not a Launch defect.
- Related OPTIONS/preflight cases: [password protection blocks OPTIONS](launch-platform-limitations-deploy-hook-secret-logs-password-protection-options.md); OPTIONS preflight was also a prior incident in [the OAuth 520 entry](nextjs-oauth-simple-authorize-state-520.md).

## Status

- **Resolved** — root cause was the app **not handling the CORS preflight `OPTIONS`** (and other unhandled verbs like `PATCH`); adding OPTIONS handling fixed the 100s response times. (Store token in repro headers `<redacted>`.)
