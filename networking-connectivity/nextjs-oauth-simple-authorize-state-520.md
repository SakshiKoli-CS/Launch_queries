QUESTION

A Next.js app’s OAuth endpoint stopped returning a response: `GET /api/oauth/simple-authorize` with `client_id`, `redirect_uri`, `state`, and `response_type=code` (e.g. on `red-panda-commerce-v2.contentstackapps.com`). The request did not appear to reach the handler. Removing the `state` query parameter made the call succeed, which suggested Launch/Cloudflare interaction. The same routes had worked for ~1–1.5 months with no app-side change; behavior worked when tested locally.

_Keywords: 520 error, OAuth simple-authorize, state param, request not reaching handler, removing state works, extra redirects, worked locally, sudden break no change, Next.js redirects, CF1003, OPTIONS preflight._

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
- Internal timeline (same app, ~11 days): **23 Apr — CF1003** (app code not following web standards); **17 Apr — OPTIONS preflight** not handled properly (**resolved via app fix**); **28 Apr — `state` param delay/not working** (investigated jointly). Customer reported the **current issue as resolved** after Launch support engagement.

**Pattern note — "sudden breaks, no changes from our side"**

- The customer flagged **three sudden issues in ~a week** on the same app, all "working for months then suddenly broke." Important framing: the **first two were resolved by application-code fixes** (CF1003 standards compliance, OPTIONS preflight handling), so a "nothing changed on our side" report does **not** rule out an app-side cause — platform/edge behavior can surface latent app issues (non-standard requests, redirect chains, slow downstream calls) that previously happened to pass.
- A **520 (not 403)** is the key signal that this is **origin not completing a valid response** (timeout/redirect-chain/latency), **not** a new Cloudflare WAF/rule block.

## Generalized guidance (for future similar queries)

- **"OAuth/API request returns 520 / doesn't reach the handler; removing `state` (or another param) makes it work"** → 520 = **origin didn't return a valid response** (timeout/reset/incomplete), not a WAF block (that'd be 403). Look at the **handler + downstream auth/session** latency and **redirect chains**, not new Cloudflare rules.
- **"It worked for months with no changes"** → not proof it's platform-side; on this app the prior two such incidents were **app-code fixes** (CF1003, OPTIONS). Check standards compliance, redirect count, env/config, and slow dependencies.
- **Recurring sudden issues on one app** → review for a common app-side root (non-standard HTTP behavior, extra redirects, latency) before attributing to Launch/Cloudflare; offer a joint call to debug live.
