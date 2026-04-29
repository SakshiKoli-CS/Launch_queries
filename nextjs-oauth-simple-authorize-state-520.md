QUESTION

A Next.js app’s OAuth endpoint stopped returning a response: `GET /api/oauth/simple-authorize` with `client_id`, `redirect_uri`, `state`, and `response_type=code` (e.g. on `red-panda-commerce-v2.contentstackapps.com`). The request did not appear to reach the handler. Removing the `state` query parameter made the call succeed, which suggested Launch/Cloudflare interaction. The same routes had worked for ~1–1.5 months with no app-side change; behavior worked when tested locally.

ANSWER

**What we observed on the platform**

- The failure showed **HTTP 520** (Web server is returning an unknown error). That means Cloudflare could reach the edge, but the **origin did not return a valid HTTP response** (timeout, connection reset, or failure before the response finished). A **WAF or rule block** would more often appear as **403** (or similar), not 520—so this was **not** attributed to new Cloudflare/Launch rules from that signal alone.
- The path **`GET /api/oauth/simple-authorize`** is handled at the origin and flows through **downstream auth/session** work. Latency or errors anywhere in that chain can surface as a 520.

**What the customer observed**

- Requests **sometimes** went through but were **slow** (confirmed in internal logs).
- In the failing path, **`api/oauth` had two additional redirects** compared to a working flow. That extra redirect chain contributed to the flow **not completing** and errors appearing—whereas the **same routes worked until recently** and **worked locally**.

**Application / framework angle**

- **Next.js** can add complexity here: the framework has **known rough edges around redirects** in API/OAuth-style flows. Combined with **how `state` is handled** in the OAuth handler and dependencies, that can interact badly with **latency and multiple redirects**, increasing the chance of timeouts or incomplete responses—what shows up upstream as **520**.

**Remediation and follow-up**

- Verify **OAuth handler** and **dependent services**, especially **`state`** handling, env vars, and config for that API.
- Internal timeline (examples from the thread): incidents including **CF1003 / standards**, **OPTIONS preflight** (fixed on app side), and **state param delay**—the last was **investigated jointly** (call/scheduling mentioned). Customer reported the **current issue as resolved** after Launch support engagement.
