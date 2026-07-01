QUESTION

Customer **Bibby Financial Services** (SF case **47071**) hits a **CSP console error**: their **nonce-based CSP** (with **`strict-dynamic`**) is **blocking Cloudflare's email-protection decode script**, so `mailto:` links (e.g. a "Contact our press office" button) are broken. Per Cloudflare docs, Cloudflare *should* auto-parse the CSP response header and attach the nonce to its injected scripts — but it isn't working. **Constraints: do NOT disable email protection** (must stay enabled), and they **cannot use host-based allowlisting** because they need `strict-dynamic` for other functionality. All their *other* nonce-based scripts (e.g. GTM) load fine. CSP is generated in Next.js **`middleware.ts`**.

_Keywords: CSP console error nonce strict-dynamic, Cloudflare email protection decode script blocked, do not disable email protection, cannot use host-based allowlist need strict-dynamic, script-src script-src-elem missing directive, nonce interpolation incorrect middleware.ts nextjs, external resource allowlist CSP, other nonce scripts load fine GTM, content-security-policy-example repo, mailto broken cdn-cgi._

ANSWER

## Root cause — the customer's own CSP was too restrictive + nonce handling was off (NOT a Cloudflare bug)

- Despite the initial framing that Cloudflare might be failing to attach the nonce, the actual cause was on the **customer's side**: **restrictive CSP directives** and **incorrect nonce handling** in their `middleware.ts`.
- Two concrete gaps:
  1. **Missing `script-src-elem` directive.** Setting only `script-src` isn't enough — modern policies need **both `script-src` and `script-src-elem`**; without `script-src-elem`, element-loaded scripts (including Cloudflare's injected decode script) get blocked even when `script-src` looks correct.
  2. **Incorrect nonce interpolation** and **missing external-resource allowlists** in the policy string.

## Fix — correct the CSP so it works with `strict-dynamic` (email protection stays ON)

- **Update the CSP to include:**
  - **proper external-resource allowlists**,
  - **correct nonce interpolation** (nonce set in the **HTTP response header**, not a `<meta>` tag — Cloudflare only reads header nonces),
  - **both `script-src` AND `script-src-elem`** directives.
- This keeps **`strict-dynamic`**, keeps **email protection enabled**, keeps full **Next.js** functionality, and lets Cloudflare's decode script load with the nonce → `mailto:` links work.
- Reference implementation: **`content-security-policy-example`** GitHub repo (correct nonce + directive setup for Next.js).

## Why the misleading signals

- **"All other nonce-based scripts (GTM) load fine, only the email-protection script is blocked"** → looked like a Cloudflare-specific parsing bug, but the real difference was that the missing **`script-src-elem`** / interpolation gap happened to block the Cloudflare-injected element-script while inline/other patterns still passed under `script-src`.
- Cloudflare **does** support this — but **only via nonces set in HTTP response headers** (not `<meta>` tags), and it needs a **cleanly-formatted CSP** to parse.

## Generalized guidance (for future similar queries)

- **Nonce-based CSP with `strict-dynamic` blocking a script (Cloudflare email-protection decode, or any injected script)** → first **audit the customer's own CSP** before blaming Cloudflare:
  - set the nonce in the **HTTP response header**, **not** a `<meta>` tag (Cloudflare only honors header nonces);
  - include **both `script-src` and `script-src-elem`** (a common miss — `script-src` alone blocks element-loaded scripts);
  - ensure **correct nonce interpolation** and the needed **external-resource allowlists**;
  - keep the CSP **cleanly formatted** so Cloudflare can parse and inject the nonce.
- **`strict-dynamic`** means you **can't rely on host-based allowlisting** (`cdn-cgi`, etc.) — you must use **nonces/hashes**, which makes the header/directive correctness above essential.
- **Don't disable email protection** to "fix" this — the correct CSP lets the injected script run with the nonce. (Disabling isn't even possible per-project — it's zone-level; see related.)
- Reference: **`content-security-policy-example`** repo for a correct Next.js CSP + nonce setup.
- Related: [Cloudflare email protection can't be disabled per project → `email_off` workaround (Concord)](launch-cloudflare-email-protection-csp-nonce-mailto.md), [Cloudflare email obfuscation `/cdn-cgi/` 404 crawler](launch-cloudflare-email-obfuscation-cdn-cgi-404-crawler.md), [500 content-length mismatch from middleware headers](launch-500-content-length-mismatch-headers-sent-middleware-downstream.md).

## Status

- **Resolved.** Cause was **the customer's own restrictive CSP + incorrect nonce handling** (not a Cloudflare bug). Fixed by updating the CSP to include **proper external-resource allowlists, correct nonce interpolation, and both `script-src` and `script-src-elem`** — email protection **stayed enabled**, `strict-dynamic` preserved, full Next.js functionality maintained. Pointed the customer to the **`content-security-policy-example`** GitHub repo. Shared with the customer and confirmed.
