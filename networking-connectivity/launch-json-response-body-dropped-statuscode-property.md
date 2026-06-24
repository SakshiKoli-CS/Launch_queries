QUESTION

Customer **Sophos** (Org `blt7eaaa4a78adf9602`, AWS NA, SF case **00057863**, **P1**) reported **form submissions failing** because the **JSON response body was not being returned to the front-end** for certain AJAX requests.

Precise repro (isolated by the customer):
- The body is dropped **only when the JSON response contains a top-level `statusCode` property AND the total number of top-level properties is fewer than 7.**
- **No `statusCode`** → body of any size returns fine. **≥ 7 top-level properties** → body returns fine.
- The **HTTP status code (200, 422, etc.) is always correct** — only the **body** goes missing.
- **Not reproducible locally**; persists **without** Next.js `proxy.js` or Launch Edge Functions; reproduced on a **minimal Express.js** app on a separate Launch instance → points to **Launch platform/runtime**, not the app.

Echo-endpoint illustration: the server adds `statusCode` + `message`, so a request body of **4 props → 6 total → empty body**; **5 props → 7 total → body returned**.

_Keywords: JSON response body missing, response body empty, statusCode property dropped, fewer than 7 properties, AJAX response empty, form submission failing, status code correct but no body, Launch runtime response bug, CL-3838, CL-3827._

ANSWER

## Cause — Launch platform/runtime bug stripping certain JSON response bodies

- Confirmed **Launch platform/runtime defect**: response bodies were dropped when the JSON had a **top-level `statusCode`** and **< 7 top-level properties**. The status code was unaffected — only the **body** was stripped before reaching the client.
- Not an application issue — reproduced on a **minimal Express app** with no Next.js/proxy/Edge Functions involved; server logs showed **no errors** and deployments succeeded, and platform-side the requests completed **200 OK** (the body loss happened in response handling, which is why "200 with empty body" was the signature).
- Tracked internally as **CL-3838 / CL-3827**.

## Resolution

- The team **identified and fixed** the issue and **deployed to production**.
- **Post-fix:** there is **no need to avoid any specific property names** (like `statusCode`) or particular response shapes/sizes going forward — the restriction is gone. Customer confirmed testing looks good.

## Generalized guidance (for future similar queries)

- **"API returns 200 but the response body is empty/missing (intermittently, by shape)"** → suspect a **platform-side response-handling issue**, not the app — especially if it's **not reproducible locally** and the **status code is correct** while only the body is lost.
- **Strong isolation signal:** reproduce on a **minimal app (e.g. Express)** with proxy/Edge Functions removed. If it still drops the body, it's **platform/runtime**, not framework/middleware.
- This specific bug (**`statusCode` + < 7 top-level props → empty body**) is **fixed** (CL-3838/CL-3827); no property-name avoidance needed. If a similar shape-dependent body-stripping recurs, escalate as a platform runtime bug with a minimal repro.

## Status

- **Resolved** — fix deployed to production; no response-shape/property-name restrictions remain; customer confirmed. (Minimal repro app was provided behind basic auth — credentials `<redacted>`.)
