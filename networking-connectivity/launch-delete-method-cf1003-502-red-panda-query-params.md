QUESTION

Customer reported that a **Next.js project (Red Panda Commerce)** on **Launch (NA AWS)** had **DELETE API requests suddenly failing today**, while **GET/POST and other methods on the same routes kept working**. The APIs had worked for a long time until today. Response:

```
code: "CF1003"
instance: "/api/stores/eb83e6ab-6c63-42b3-8f44-511c992130ea/attributes"
message: "Service temporarily unavailable. Please try again."
request_id: "30660938-c8e4-40f4-9a6e-aeff74dcd7ae"
status: 502
title: "Upstream Service Error"
type: ".../troubleshooting-launch-response-error-codes#bad-gateway-cf1003"
```

Their extensive app logging showed the **DELETE requests weren't even reaching the handler** — rejected by Launch/Cloudflare upstream. Follow-up: "but how was it working perfectly till the day before yesterday?"

_Keywords: CF1003, 502, Upstream Service Error, DELETE failing, DELETE with body, GET POST work DELETE doesn't, not reaching handler, RFC 7231, query params, worked until recently, Red Panda Commerce._

ANSWER

## Cause — DELETE request with a body (not supported)

- **DELETE requests without a body work fine; DELETE requests that include a body** can fail with **CF1003 (502)** and **may not reach the handler.**
- Per **RFC 7231**, a **payload on a DELETE has no defined semantics**, so it's **not consistently supported** across HTTP infrastructure (proxies/gateways/CDNs). To align with standard web practices, **Launch does not support payloads on GET or DELETE.**

## "But it worked until the day before yesterday?"

- Because **DELETE + body is not portable**, it **may or may not work** depending on the upstream path — it can appear to work for a while, then start failing, **even with no application change**. "It worked before" doesn't make it supported.

## Fix — move the payload to query parameters

- Don't send a body with DELETE; pass identifiers/filters as **query parameters**, e.g.:
  ```
  DELETE /api/stores/{id}/categories?categoryIds=<id>
  ```
  (adapt to your API). Or refactor to a method that clearly supports a body (e.g. POST for a "delete by payload" action).

## Generalized guidance (for future similar queries)

- **"DELETE fails with CF1003/502 but GET/POST work on the same route"** → almost always a **DELETE with a body**; Launch rejects GET/DELETE payloads upstream (RFC 7231) so it never reaches the handler. Move data to **query params**.
- **"It worked until recently, no code change"** → DELETE+body is **not portable** and only *intermittently* passes; not a regression to chase — remove the body.
- **CF1003 with no DELETE body** → check **RFC 3986** path/query (malformed/encoded URL) issues per the troubleshooting doc.

## Status

- Explained the DELETE-with-body limitation and the **query-parameter** fix; customer acknowledged.
