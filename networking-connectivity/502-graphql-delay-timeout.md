QUESTION

**Sophos** (org `blt7eaaa4a78adf9602`, project `68be5c7df1773c96089c2859`, SF **00056063**) reported three concurrent issues:

1. **Brief outage** (~3:20–3:30 AM PT) — site unavailable for ~10 minutes; server logs showed no errors during that window
2. **502 errors** on `https://www.sophos.com/en-us/products/endpoint-security` and its preview counterpart `sophos-preview.contentstackapps.com/en-us/products/endpoint-security` — prod recovered without changes; preview persisted longer; same codebase used by a third instance was unaffected throughout
3. **Sporadic 503 (Apollo errors)** — red toast messages in the Launch server management UI when viewing server logs or redeploying a commit

The page is a Next.js SSR page that fetches data from Contentstack via GraphQL to render multiple components. Logs also showed an `ENVIRONMENT_FALLBACK` error from `next-intl`.

ANSWER

**Issue 1 — Brief outage (3:20–3:30 AM PT)**

No platform-level incident or infrastructure event was identified during that window. Server logs for the affected instance were unavailable (obscured by noise from a separate ongoing investigation). **Root cause not confirmed** — unresolved in thread.

**Issue 2 — 502 errors (resolved)**

**Cause: app response exceeded Launch's 30-second timeout.**

Launch returns a **502** when the origin application does not respond within **30 seconds**. Sophos had recently introduced a GraphQL rate-limit optimization that **intentionally delayed requests** when the `x-ratelimit-remaining` response header fell below a configured threshold. On pages with many sections (each requiring GraphQL calls), these cumulative delays exceeded 30 seconds — particularly after **cache purges across multiple pages**, when all sections had to fetch fresh data concurrently.

Key diagnostic points:
- GraphQL API itself was not failing — Sophos's own logger showed no GraphQL errors
- The `ENVIRONMENT_FALLBACK` error from `next-intl` appeared in logs but was a **red herring** — it surfaced across all servers while pages rendered correctly; customer confirmed it was not the cause
- The preview instance (`sophos-preview`) persisted longer because its **lower traffic** meant fewer concurrent requests naturally resolving the backlog — same underlying cause, different recovery pace
- A third instance using the same codebase was unaffected because it was presumably hitting a different cache state or traffic pattern

**Fix:** Customer reverted the rate-limit delay optimization. Issue resolved.

**Suggested checks for similar 502s:**
- Confirm from Launch logs whether the origin timed out (no response within 30s) vs. the app returning an error response
- Check if the page performs many sequential or semi-parallel data fetches — each adds latency that compounds under cache-miss conditions
- Any app-side throttling logic (rate-limit headers, sleep/delay before retries) should be validated against worst-case page complexity and cache-miss scenarios

**Issue 3 — Apollo 503 in Launch management UI**

Sporadic 503 errors in the Launch UI (server log viewer, redeploy actions) — observed in Chrome. Investigation was ongoing when the 502 issue resolved and the thread was closed. **Not resolved in thread.**

**Load testing — follow-up clarification (Eight25Media / Sophos)**

Customer asked what happens after notifying Launch of a planned load test, and whether it would isolate environments or increase rate limits:

- **Notifying Launch** is required so the test traffic is recognized as **legitimate and not flagged/blocked as malicious** — it does not result in environment isolation or platform adjustments
- **CDA rate limits are org-specific** — load testing against the GraphQL CDN layer does not create significant load on the Contentstack backend; it should not affect uncached production endpoints like form submissions
- A **temporary rate-limit increase is not provided** for load testing within existing limits — if the test is intended to validate 429 fixes, it must be run within the existing quota
- Reference: [Load testing docs](https://www.contentstack.com/docs/developers/launch/load-testing)
