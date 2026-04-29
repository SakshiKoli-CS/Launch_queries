QUESTION

A Next.js app (Red Panda Commerce) on **Launch** in **NA AWS** reported that **DELETE** API calls began failing **while GET/POST/other methods on the same routes continued to work**. The failure showed **502** with Launch error **CF1003** (“Upstream Service Error” / “Service temporarily unavailable”), link: [Bad gateway / CF1003 troubleshooting](https://www.contentstack.com/docs/developers/launch/troubleshooting-launch-response-error-codes#bad-gateway-cf1003)—example path **`/api/stores/{storeId}/attributes`**. Internal logging suggested **DELETE traffic was not reaching the application handler**, as if rejected **upstream** (Launch / Cloudflare) rather than failing inside the route code. The APIs had behaved as expected historically until the report date.

ANSWER

**Cause (platform behavior)**

- **DELETE requests without a request body behave as expected.**
- **DELETE requests that include a body** can fail with **CF1003** and **502**, and **may not reach the handler**. This aligns with **[RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231)**—a **payload on DELETE has no defined semantics**, so behavior across proxies, gateways, and CDNs is **inconsistent**.
- To align with common web practices, **Launch does not support payloads on GET or DELETE.**

**Customer follow-up (“it worked until recently”)**

- Because **DELETE + body is not portable** across HTTP infrastructure, behavior **may appear to work** until a proxy, CDN, or gateway path changes—even when **application code did not.**

**Recommended approach**

- **Do not send a body with DELETE.** Pass data via **query parameters** (and/or refactor to another method if your API design requires a body). Example pattern:  
  `DELETE /api/stores/{id}/categories?categoryIds=<id>`  
  (adapt query names to your API.)

The public **[CF1003](https://www.contentstack.com/docs/developers/launch/troubleshooting-launch-response-error-codes#bad-gateway-cf1003)** troubleshooting page also lists **RFC 3986** path/query issues as a cause of upstream rejection—rule those out if DELETE has **no body** and still fails.

**Suggested checks**

- Confirm whether failing DELETEs include a **body**; retest **DELETE without a body** for the same route.
- If a body was required for the operation, move identifiers or filters to **query string** or use a method that clearly supports a body (e.g. **POST** for a “delete by payload” action), per your API conventions.
