QUESTION

Customer **Next** (SF case **00058746**, urgent — blocking go-lives) reported a **long-term caching / "drift" issue**: when content is changed and published, **cache clears don't seem to happen as expected**. Their architect's monitoring suggests the **Production Launch cache persists longer than the specified 60 seconds** — possibly held for up to **two hours**.

Cache settings across stacks:
- **Staging:** `no-store, no-cache`
- **Production:** `public, max-age=0, s-maxage=60s`

They asked whether the cache is clearing correctly after 60 seconds or being held longer. (Next International Stack key `blt9c9c23f853325239`, delivery token `<redacted>`, production-desktop environment.)

_Keywords: cache not clearing, cache drift, stale content after publish, s-maxage persisting too long, s-maxage=60s non-standard, Cache-Control format, max-age=0 s-maxage, CDN cache expiry, two hours cache._

ANSWER

## Likely cause — non-standard `s-maxage` value (`60s` instead of `60`)

- The Production header was set to **`public, max-age=0, s-maxage=60s`**.
- **`s-maxage` should be an integer number of seconds only** per the HTTP `Cache-Control` standard — i.e. **`s-maxage=60`**, not `s-maxage=60s`.
- **`60s` is a non-standard format** and **may be parsed inconsistently** by intermediate caches/CDNs, which can lead to the **unexpected cache-persistence / "drift"** behavior the customer observed.

## Recommended fix

- Update Production to the **standard format**:
  ```
  public, max-age=0, s-maxage=60
  ```
- Then **monitor cache expiry again**; once updated, the behavior can be **re-validated** from the Launch side. (Ref: Cache-Control Header docs.)

## Note on scope

- With the delivery token + stack key in hand, the initial read was that this leans **CMS/cache-configuration**, not a Launch infrastructure fault — the header value itself is the first thing to correct before deeper investigation.

## Generalized guidance (for future similar queries)

- **"Cache persists longer than my `s-maxage` / content doesn't refresh after publish"** → First **check the `Cache-Control` value format**. Use **integer seconds** (`s-maxage=60`), never a unit suffix (`60s`, `1m`) — non-standard values are parsed unpredictably by CDNs and can cause over-long caching.
- Correct the header, then re-measure expiry before assuming a platform cache bug.
- `max-age=0, s-maxage=N` means browsers always revalidate while the **shared/CDN cache** serves for `N` seconds — confirm the intended `N` and its (integer) format.

## Status

- Customer asked to **update `s-maxage=60s` → `s-maxage=60`** and re-monitor; Launch to re-validate expiry afterward. Awaiting customer confirmation at last update.
