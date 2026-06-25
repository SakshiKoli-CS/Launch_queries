QUESTION

Some users intermittently get a **request-size error (CF1001, "Request body exceeds size limit")** when accessing `https://www.contentstack.com/contentcon-europe`. The reporter **couldn't reproduce it** and confirmed the **page size is well under the 5 MB limit**. The error later appeared **across multiple users, devices, and geographies**, and then on **Docs entry pages** too (e.g. `/docs/personalize`). The project uses **Contentstack Personalize via an Edge Function**.

_Keywords: CF1001 request body exceeds size limit, 5MB limit, 413 error, large request headers cookies, header size 9KB limit, Edge Function Personalize modifying requests, HAR file investigation, misleading error body vs header, docs page down, CDA response size 50MB, max_response_size 70MB, skip limit batches._

ANSWER

This thread had **two separate root causes** that both surfaced as a size error — don't conflate them.

## Root cause A — request HEADER size (~9 KB), not body size (the CF1001 case)

- The failing call was a **GET** request. Investigation (via a **HAR file** from an affected user — needed to inspect total request size, **cookie size**, headers, query params) showed the issue was **not** the **request body / 5 MB limit** at all.
- It was the **request HEADER size**, which must stay around **~9 KB**. Some users had **large/accumulated cookies** (and/or **Edge Function Personalize modifications adding headers/cookies**) pushing the **headers over ~9 KB** → the request failed.
- **Misleading error:** Launch returned the **request-body-size error (CF1001)** even though the real cause was the **header limit**. (Launch is improving this to report the **header limit** accurately. Docs for the body limit: [CF1001 request-body-exceeds-size-limit](https://www.contentstack.com/docs/developers/launch/troubleshooting-launch-response-error-codes#request-body-exceeds-size-limit-cf1001).)
- **Fix:** **reduce request header size** — audit **cookies** and any **Edge Function / Personalize logic that adds headers or cookies**; trim accumulated cookies. Bringing headers under ~9 KB resolves it.

## Root cause B — CDA RESPONSE size > 50 MB on Docs pages (separate issue, "site down")

- The later **Docs entry-page failures** (e.g. `/docs/personalize`, fetched via the **Delivery SDK / CDA**) were a **different** problem: the **CDA response exceeded the 50 MB response-size limit.**
- Trigger: fetching **a single entry with a large amount of included/referenced data** — no code change, but the **Docs team had published more content**, inflating the response.
- **Fixes:**
  - **Fetch in batches** using **`skip` + `limit`** instead of pulling everything at once (recommended — keep responses small).
  - Or **raise the CDA response-size limit** from **50 MB** via a **KCR request** setting **`max_response_size` to `70000000` (70 MB)** (Super Admin plan). Use sparingly; smaller requests are preferred.

## Generalized guidance (for future similar queries)

- **CF1001 / "request body exceeds size limit" but the page/body is clearly small** → suspect **request HEADER size (~9 KB)**, not the body. Common cause: **bloated/accumulated cookies** or an **Edge Function/Personalize layer adding headers**. The CF1001 message can be **misleading** — it may actually be a header-limit hit. **Get a HAR** to confirm header + cookie size.
- **Intermittent + only some users** is a tell for **header/cookie bloat** (varies per user/session) rather than a fixed platform issue.
- **"Docs/site down" with a size error from CDA** is a **different limit**: the **50 MB CDA response-size limit**. Fix by **batching with `skip`/`limit`** (preferred) or raising **`max_response_size`** (KCR, e.g. 70 MB). Watch for **content growth** (more published/referenced entries) inflating a previously-fine single-entry fetch even with no code change.
- **Key distinction:** **request header limit (~9 KB)** and **request body limit (5 MB)** and **CDA response limit (50 MB)** are three different ceilings — match the symptom to the right one.

## Status

- **Two causes identified.** (A) The CF1001 errors were due to **request header size (~9 KB)** from **large cookies / Edge-Function Personalize header modifications**, not the body/5 MB limit — Launch improving the error to name the header limit; customer to **trim headers/cookies**. (B) The **Docs page failures** were a **CDA 50 MB response-size limit** from a large single-entry-with-includes fetch (content growth) — fix via **`skip`/`limit` batching** or a **KCR `max_response_size` 70 MB** increase. Customer actioning both; **open/monitoring**.
