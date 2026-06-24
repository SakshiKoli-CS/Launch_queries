QUESTION

Customer (contentstack.com on Launch) reported repeated **CF1004 — "Request Headers Too Large" (HTTP 413)** errors, including on staging — frequent enough that they browse in **incognito** to avoid it. They had already **capped cookies at 8 KB** and didn't understand why requests were still rejected.

Questions:
1. What was the **total request header / cookie size** for the rejected request?
2. Does Launch **log the actual size** on a CF1004 rejection?
3. Using request ID `36a88e23-c58e-4629-ab8e-5f89f9a84f3a`, can the exact header/cookie size that caused the rejection be traced?
4. Is the limit applied **only to the Cookie header**, or to the **combined total of all request headers**?

_Keywords: CF1004, 413 Request Entity Too Large, Request Headers Too Large, total request header size, cookie size limit, 8KB header limit, AWS Lambda header limit, RequestEntityTooLargeException, strip headers at edge, incognito avoids error, Observe origin logs._

ANSWER

## The limit is on TOTAL request headers — not just the Cookie header

- **This is the key point:** the size limit applies to the **combined total of all request headers** (request line + every header), **not just the `Cookie` header.** So capping cookies at 8 KB isn't enough if **other headers** push the total over — which is exactly what happened here.
- The request failed with **413 (RequestEntityTooLargeException)** at the origin — **rejected by AWS Lambda** (the serverless origin's header limit). Launch returning **CF1004 / 413** is **expected behavior** when the request exceeds that limit.

## Actual measured sizes (yes, it's logged)

- The rejected request (request ID `36a88e23-...`) was **8,857 bytes (~8.6 KB)** total. Since it was a **GET**, that size is **entirely request line + headers** (no body).
- Over the prior 30 days, the **smallest request that failed was 8,568 bytes (~8.4 KB)**.
- The size **is logged** — see `network.bytes_read` / `serverless_request_in_size` in the Observe origin-request log for the request ID.

## The limit is a range (~9 KB), not a hard line

- The effective limit is **approximately 9 KB** for total request headers, but it's **not strict** — some requests slightly above may still pass, while others fail, causing **intermittent** rejections.
- **Recommendation:** treat **8 KB as the safe upper bound for total request headers** (all headers combined, not just cookies) to avoid intermittent CF1004s. Guidance may be revised if a higher consistently-safe limit is validated.
- (Header size limits like this are **under-documented** across platforms; Contentstack noted intent to document the Launch header limit more clearly.)

## Fixes

- **Reduce total header size:** remove unnecessary cookies/headers no longer needed.
- **Strip unnecessary cookies/headers at the edge** via a **Launch Edge Function** before the request reaches the origin — this also **unblocks existing users who've accumulated large cookies** (which is why incognito, with no cookies, avoids the error).
- **Diagnose:** use **Observe** origin-request logs (filter by request ID; check `network.bytes_read`), and/or add **edge-function logging** to measure header size in real time.

## Generalized guidance (for future similar queries)

- **"CF1004 / 413 Request Headers Too Large, but our cookies are under 8 KB"** → the limit is on the **TOTAL of all request headers**, not just `Cookie`. Sum all headers + request line; non-cookie headers can push it over.
- **Limit ≈ 9 KB (AWS Lambda origin), not strict** → use **8 KB total headers** as the safe ceiling; expect **intermittent** failures near the boundary.
- **Incognito "fixes" it** → because there are no accumulated cookies; the real fix is **trimming headers/cookies**, ideally **stripped at the edge** via an Edge Function to unblock users who already have bloated cookies.
- **The rejection size IS logged** — `network.bytes_read` / `serverless_request_in_size` in Observe per request ID.

## Status

- Explained: total-headers limit (~9 KB, use 8 KB safe), measured sizes from logs, edge-strip fix. After follow-up, **no further 413/CF1004 errors observed** in Observe or server logs over the subsequent week.
