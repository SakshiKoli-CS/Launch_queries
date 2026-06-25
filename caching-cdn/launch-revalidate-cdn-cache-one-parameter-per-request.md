QUESTION

Customer **Reply** (for Moncler, SF case **00053998**) is using the **Launch Revalidate CDN Cache API** and wants to **revalidate a specific path under a specific hostname** in one call by combining `hostnames` + `cachePath`. Their payload **fails**:

```json
{
  "hostnames": ["development.monclergroup.com"],
  "cachePath": { "path": "/pippo", "isPrefix": false }
}
```

The docs show `hostnames` and `cachePath` can be passed separately — is combining them supported?

_Keywords: Revalidate CDN Cache API, cachePath hostnames cacheTags, combine parameters one request, payload failing, revalidate path under hostname, one revalidation strategy per request._

ANSWER

## No — only ONE parameter per revalidation request

- CDN revalidation supports **one parameter per request** — **either `cachePath`, `hostnames`, OR `cacheTags`** — **not a combination.**
- **Combining `cachePath` and `hostnames`** (or any two) in a single payload **fails**: the API accepts **only one revalidation strategy per request**.
- Therefore **revalidating a specific path under a specific hostname in a single call is not currently possible** — you'd need **separate requests** per your requirement.
- (More granular control is noted as an **enhancement request** for the revalidation endpoint.)
- Docs: Launch API → Revalidate CDN Cache.

## Generalized guidance (for future similar queries)

- **"Can I combine `cachePath` + `hostnames` (or other params) in one Revalidate CDN Cache call?"** → **No** — exactly **one** of `cachePath` / `hostnames` / `cacheTags` per request; a multi-parameter payload **fails**.
- **Path-scoped-to-a-hostname in one call isn't supported** today → split into separate requests; the combined-granularity is a pending enhancement.
- Pick the **single strategy** that best fits (e.g. `cacheTags` to invalidate many URLs in one request — see [cache priming](launch-cache-priming-failures-rate-limits-batching.md) / [ISR cap](launch-isr-support-cache-revalidation-limit-cap.md) entries).

## Status

- Clarified: **one parameter per request**; combining `cachePath` + `hostnames` is **not supported** (use separate calls). Logged as an **enhancement request**.
