QUESTION

A developer calling the Launch GraphQL API (`/manage/graphql`) for cache invalidation (`revalidateCDNCache`) asked about the expected HTTP status code and response format. The code checked `response.status === 200` to determine success and threw on non-200. A screenshot from Contentstack Automate showed the "Revalidate CDN Cache" step with a **green success checkmark** alongside an error body:

```
errors[0].message: Ratelimit Exceeded Error
errors[0].path[0]: revalidateCDNCache
errors[0].extensions.code: INTERNAL_SERVER_ERROR
data: (empty)
```

The question: why does a step show success when there is clearly an error in the output?

ANSWER

**HTTP status code**

The Launch GraphQL API (`/manage/graphql`) returns **HTTP 200 OK** for almost all responses — including application-level errors such as rate limit exceeded, invalid input, or unauthorized. Non-200 responses only occur for infrastructure-level failures (gateway errors, authentication failures at the proxy layer, malformed requests).

**Response format**

Responses follow the standard GraphQL JSON envelope:

| Outcome | Shape |
|---|---|
| Successful mutation | `{ "data": { "revalidateCDNCache": { ... } } }` |
| Application error | `{ "data": null, "errors": [{ "message": "...", "extensions": { "code": "..." } }] }` |
| Mixed (partial data + errors) | `{ "data": { ... }, "errors": [...] }` |

**Why the Automate step shows green despite an error**

Contentstack Automate evaluates step success based on **HTTP status code only** — it received 200, so it marks the step as succeeded (green checkmark). The `errors` array in the response body is not automatically inspected. The annotation in the screenshot is correct: it is a success at the transport level, but the mutation itself failed.

**Implication for integrations and automation**

When calling the Launch GraphQL API programmatically or via Automate, always inspect the response body for an `errors` field — do not rely on HTTP 200 alone:

```js
const result = await response.json();
if (result.errors && result.errors.length > 0) {
  // handle application-level error
  throw new Error(result.errors[0].message);
}
return result.data;
```

In Automate, add a conditional step after the "Revalidate CDN Cache" action that checks the output for the presence of `errors` if you need to branch on failure.

**Rate limit error specifically**

The `INTERNAL_SERVER_ERROR` code with "Ratelimit Exceeded Error" message means the revalidation request was rejected because the org has hit its cache revalidation limit (2,000 requests per rolling 24-hour window by default). Check usage in the Launch analytics dashboard; the limit is negotiable — contact support if the cap is too low for the workflow.
