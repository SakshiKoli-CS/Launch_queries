QUESTION

**iODigital** (org `blt8e1dd75147bb4b66`, project `69b16c0cab45d0bff3aea12c`, AWS EU; SF **00056249**) reported that **HTTP response streaming via `Transfer-Encoding: chunked`** is not working on Launch. Their Next.js API route streams responses to the frontend incrementally, but the response is returned all at once instead of appearing progressively in the browser.

Demo: `https://streaming-test.eu-contentstackapps.com/`

They asked whether Launch supports HTTP streaming / chunked transfer encoding, or whether responses are buffered at the platform or edge layer.

ANSWER

**Launch currently buffers responses — HTTP streaming to the client is not supported.**

Launch buffers the full response before delivering it to the client. Next.js streaming features that rely on the response being sent incrementally to the browser — including:

- `Transfer-Encoding: chunked` API routes
- Server-Sent Events (SSE)
- React Server Components (RSC) streaming
- `ReadableStream` / `stream` responses

— will appear to return all at once rather than progressively. The buffering happens at the platform/edge layer, not in the application code.

**Distinction from CTE support for upstream CDN connections**

Launch's Node.js/Next.js runtime does use `Transfer-Encoding: chunked` at the HTTP/1.1 level when communicating with an upstream CDN (e.g. Akamai) — this is about the origin→CDN protocol leg and is unrelated to end-user streaming. See: [Launch origin CTE for Akamai HTTP/2](chunked-transfer-encoding-akamai-http2.md).

End-to-end streaming from the Launch origin to the browser is the unsupported case documented here.

**Status**

Feature request raised — platform team informed. ETA not confirmed at time of thread; no documentation existed confirming or denying streaming support prior to this investigation.

**Suggested checks for similar reports**

If a customer reports that a streaming Next.js route (SSE, RSC streaming, chunked API response) shows the full response at once rather than progressively — this is the expected current behavior on Launch, not an app bug. Confirm the feature request is logged and share the ETA once available.
