QUESTION

Customer **Concord** has multiple **edge URL redirects** configured in **`launch.json`**. They work, **except query strings aren't preserved** — critical for their client's **UTM parameters** (`utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content`). Following the [Edge URL Redirects docs](https://www.contentstack.com/docs/developers/launch/edge-url-redirects), they can capture specific params via regex, but there's **no example for reusing a captured param as a param in the destination** or **preserving the entire query string**. Worse: **any time they put a `?` in the destination pattern, the deployment fails.**

Example goal: redirect `/newsroom-detail/(.*)?utm_source=…&utm_medium=…` → `/newsroom?utm_source=…&utm_medium=…` (preserving all UTMs).

_Keywords: launch.json redirect query string not preserved, UTM parameters lost on redirect, preserve entire query string redirect, capture query param reuse in destination, question mark in destination fails to deploy, escaped ? in source :param syntax, named param capture utm_source :utm_source, edge function to preserve query string, statusCode 308 301 redirect._

ANSWER

## Two limitations of `launch.json` redirects here

1. **Query strings are NOT preserved automatically** on `launch.json` redirects — only the path is carried over unless you explicitly capture and re-emit each parameter.
2. **Putting a `?` in the DESTINATION causes the deployment to fail.** (In the *source* you escape it as `\\?`; but a literal `?` in the destination breaks the deploy.)

## Interim workaround (fragile) — capture each UTM by name and reconstruct it

- You can **explicitly capture each parameter in the source** (escaping the `?` as `\\?`) and **reconstruct them in the destination** with named `:param` tokens:
  ```json
  {
    "redirects": [
      {
        "source": "/newsroom-detail/:slug\\?utm_source=:utm_source&utm_medium=:utm_medium&utm_campaign=:utm_campaign&utm_term=:utm_term&utm_content=:utm_content",
        "destination": "/newsroom?utm_source=:utm_source&utm_medium=:utm_medium&utm_campaign=:utm_campaign&utm_term=:utm_term&utm_content=:utm_content",
        "statusCode": 308
      }
    ]
  }
  ```
- **Why this is fragile:** it only matches when **all** the named params are present in that **exact order**; it can't preserve an **arbitrary/unknown** query string; and the customer still reported **deploy failures whenever a `?` appears in the destination**. So this doesn't robustly solve "preserve the entire query string."

## Recommended fix — use an Edge Function instead of `launch.json` redirects

- Because **query params aren't preserved automatically** and a **`?` in the destination fails to deploy**, do the redirect in an **Edge Function** (`functions/[proxy].edge.js`) rather than the static `launch.json` redirect config.
- An Edge Function can **read the incoming URL's full query string and append it to the redirect target dynamically** — preserving the **entire** query string (all UTMs, in any combination) without enumerating each parameter and without the `?`-in-destination deploy failure.
- Refer to the **Edge Function documentation** and the **Launch Edge Function example repo** for the redirect-with-query-preservation pattern.

## Generalized guidance (for future similar queries)

- **`launch.json` redirects do NOT preserve query strings automatically**, and a **literal `?` in the `destination` breaks deployment.** For anything beyond a fixed, fully-enumerated set of params, **don't use static redirects.**
- **To preserve the entire query string (UTMs, arbitrary params) → use an Edge Function**, which reads `request.url`'s search string and appends it to the redirect `Location` dynamically. This is the robust answer whenever the param set is unknown/variable or you need the whole query string carried through.
- The named-param `:utm_source` reconstruction approach only works for a **fixed, ordered, fully-present** set of params — treat it as a stopgap, not the solution.
- Related: [launch.json rewrites/config lost on failed deployment](launch-json-rewrites-config-on-failed-deployment.md), [rewrite/proxy changes URL bar → use absolute URLs](launch-rewrite-proxy-url-bar-changes-origin-absolute-urls.md), [bulk 301 redirects at scale → edge/API route](launch-bulk-301-redirects-scalable-edge-api-route-bulk-import-cms-product.md), [edge function skipped / no valid export → rewrites broken](launch-edge-function-skipped-no-valid-export-rewrites-redirects-broken.md).

## Status

- **Answered / recommended.** `launch.json` redirects **don't preserve query strings automatically**, and a **`?` in the destination causes deploy failure**. The interim named-`:utm_*`-capture workaround is **fragile** (fixed param set, exact order, still `?`-fragile). Recommended handling the redirect in an **Edge Function** to capture and preserve the **entire query string dynamically**; pointed the customer to the **Edge Function docs + example repo**. Customer accepted.
