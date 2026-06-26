QUESTION

Can the **response body size limit** for websites deployed on Launch be **increased**? (Ref: Platform limits on Launch.) Org `blt45a2a52367b47001`, AWS NA, SF case **00052985**.

_Keywords: increase response body size Launch, response body hard limit, platform limits, 5MB response, serve large assets, Contentstack Assets, Image Delivery API, files larger than 5MB, cannot increase limit._

ANSWER

## No — the response body size is a hard platform limit (not raisable)

- The **response body size on Launch is a hard platform limit and cannot be increased.**

## If you need to return large assets — use Contentstack Assets + Image Delivery API

- If the use case is **receiving assets in the response body**, **don't try to push them through the app response.** Instead **serve them via Contentstack Assets** and the **Image Delivery API**, which supports **files larger than 5 MB** and serves static assets efficiently.
- This keeps the implementation **scalable** and sidesteps the response-body cap entirely.
- Refs: Creating and uploading assets; Image Delivery API.

## Generalized guidance (for future similar queries)

- **"Can we increase the Launch response body size?"** → **No** — it's a **hard platform limit.** Don't promise a raise.
- **For large files/assets**, **offload to Contentstack Assets + Image Delivery API** (supports >5 MB) rather than returning them in the app's response body.
- Distinguish the limits: this is the **Launch app response body** hard cap — separate from the **request header (~9 KB) / request body (5 MB) / CDA response (50 MB)** ceilings (see [CF1001 size limits](launch-cf1001-request-size-error-header-limit-9kb-vs-cda-response-50mb.md)).

## Status

- **Answered.** Response body size = **hard limit, not increasable.** For large assets, use **Contentstack Assets + Image Delivery API** (>5 MB supported). Customer satisfied.
