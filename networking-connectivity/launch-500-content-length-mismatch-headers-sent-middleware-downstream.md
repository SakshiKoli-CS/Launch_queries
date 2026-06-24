QUESTION

Customer **Beal / betterbybeal.com** (SF case **00057339**, P1 Pingdom alert) — site returning **HTTP 500 Internal Server Error**, recurring (alert auto-closed but 500s continued), site down then later back up. Customer said **no changes were made**, asked for the cause.

_Keywords: 500 Internal Server Error, RequestContentLengthMismatchError, UND_ERR_REQ_CONTENT_LENGTH_MISMATCH, content-length header mismatch, ERR_HTTP_HEADERS_SENT, cannot append headers after sent, Next.js middleware fetch failed, downstream API failure, forwarding request headers, no code change._

ANSWER

## Errors in the application logs (app-side, not Launch)

1. **`RequestContentLengthMismatchError` (`UND_ERR_REQ_CONTENT_LENGTH_MISMATCH`)** — *"Request body length does not match content-length header"* — thrown from a **`fetch` inside the app's Next.js middleware**.
   - Likely cause: the app is **copying/forwarding request headers from one request to another outbound `fetch`**, carrying over a **stale `Content-Length`** that doesn't match the new body → mismatch. Related to **request-forwarding / promise handling** in their middleware.
2. **`ERR_HTTP_HEADERS_SENT`** — *"Cannot append headers after they are sent to the client"* — the app **appends headers after the response was already sent**.
3. **`fetch failed`** in middleware → the app's calls to **downstream systems** were failing.

These are **application errors**; **no change was made on the Contentstack/Launch side.**

## "But we made no changes" — a downstream failure can cause this

- It's **not always a code change** that triggers such an outage. The logs show **failures fetching from a downstream system** — which can happen simply because that **external service (API/DB/CMS/etc.) is down or erroring**, with no app change at all.
- So "nothing changed on our side" doesn't rule out an app-level outage — **check the downstream systems the app calls.**

## Direction of the fix (app-side)

- **Don't blindly copy/forward `Content-Length`** (or hop-by-hop headers) when re-issuing a `fetch` — let the HTTP client set `Content-Length` for the new body, or strip it before forwarding. This resolves `UND_ERR_REQ_CONTENT_LENGTH_MISMATCH`.
- Fix the **`ERR_HTTP_HEADERS_SENT`** path — don't append/modify headers after the response has been sent.
- **Audit downstream calls** (APIs/DB/CMS) for failures and add resilient error handling so a downstream blip doesn't 500 the whole page.

## Generalized guidance (for future similar queries)

- **`RequestContentLengthMismatchError` / `UND_ERR_REQ_CONTENT_LENGTH_MISMATCH`** in middleware/`fetch` → the app is **forwarding a request body with a mismatched `Content-Length`**, usually from **copying headers across requests**. Strip/recompute `Content-Length` on forwarded fetches.
- **`ERR_HTTP_HEADERS_SENT`** → app appends headers after sending the response; fix the response lifecycle.
- **500s with `fetch failed` + "we changed nothing"** → check **downstream/external systems** the app depends on; a downstream outage 500s the app with no code change. Not a Launch defect.
- Offer **codebase access** to help pinpoint the middleware/forwarding bug.

## Status

- Site **recovered**; root cause traced to **app middleware errors (content-length mismatch, headers-after-sent) + downstream fetch failures**, not a Launch/Contentstack change. Customer was **reluctant to investigate further** (planning to churn; Launch only powered this one site). Investigation on Launch side **complete** — fix is app/downstream-side.
