QUESTION

Multiple customers on **Azure EU** (e.g. Molteni/Deloitte, SF case **00057455**) reported **intermittent timeout failures** where **API requests don't reach the backend**, showing **HTTP status 0** with **`ETIMEDOUT`** in origin/server logs — bursts of repeated failures within short windows, often under peak/registration load, causing production instability.

Log signature:
```
[ORIGIN] ⨯ [Error [APIError]: Request failed before a response was received (HTTP status 0)...
  { error_code: 'ETIMEDOUT', status: 0, ... digest: '4047115660' }
```

They asked whether there were origin/infra connectivity issues, throttling, or EU-region instability around the timestamps — and later, what the platform's HTTP/HTTPS connection limit is (since they also hit it on **external HTTP calls**).

_Keywords: ETIMEDOUT, HTTP status 0, request failed before a response was received, intermittent timeouts, Azure outbound connection limit, SNAT port exhaustion, outbound connections under load, HTTP keep-alive connection reuse, enforce IPv4 outbound, fetch content via SDK, external HTTP calls timing out, Azure EU._

ANSWER

## Cause — Azure outbound connection limits/timeouts under concurrent load

- The errors are **most likely Azure outbound-connection limits/timeouts under high load**, **not** an issue with the Contentstack API itself. **`status: 0` / "request failed before a response was received"** means the request **never reached the API** — platform-side API logs for the stack look **clean** (the call didn't arrive), which confirms the timeout is on the **app's outbound connection**, not the API.
- Per Azure: when **many outbound connections open simultaneously**, some requests **time out before the connection is established**, due to **SNAT port-allocation constraints / platform connection limits**. Ref: Microsoft — *Troubleshoot intermittent outbound connection errors* (App Service).
- This is **not purely about the site's peak traffic** — it's about **how Azure handles outbound connections under concurrent load**, and it applies to **any outbound call** from the app (Contentstack **SDK** fetches **and** external HTTP/HTTPS calls).

## Mitigations (effective on similar issues elsewhere)

1. **Enable HTTP keep-alive (connection reuse)** — reuse outbound connections instead of opening a new one per request, reducing port pressure.
2. **Enforce IPv4 for outbound requests** — particularly when **fetching content via the Contentstack SDK**.

These two changes have **reduced similar timeout issues in other environments**.

## Notes

- Not yet published in official Contentstack docs — these **Azure-related outbound issues are recent**, and the guidance came from the **Azure engineering team**; it's being incorporated into docs. Contentstack is also **working with Azure** (support case) + Cloudflare on the underlying behavior.
- A "small peak" can still trigger it because the constraint is **concurrent outbound connections / SNAT ports**, not raw request volume.

## Generalized guidance (for future similar queries)

- **"`ETIMEDOUT` / HTTP status 0 / 'request failed before a response was received', intermittent, under load (Azure)"** → **Azure outbound-connection (SNAT) limits**, not the Contentstack API (the request never arrives — API logs are clean). Confirm by checking that platform-side API logs show **no corresponding request**.
- **Mitigate:** **enable HTTP keep-alive (connection reuse)** and **enforce IPv4 for outbound requests** (esp. SDK fetches). Applies to **all outbound calls**, including external HTTP/HTTPS.
- **Not (just) peak load** — it's concurrent-connection/port pressure; even small peaks can trip it.

## Status

- Identified as **Azure outbound-connection timeout** behavior; recommended **keep-alive + IPv4 enforcement**. Working with Azure on root cause; guidance pending official-docs publication. Customer-reported incident resolved; recurrence mitigated via the above. (Stack `blt7a87005427cbbb8a`; delivery/preview tokens `<redacted>`.)
