QUESTION

Customer **First Interstate Bank** (Org `blt6eec7c5cd96cdb6b`, SF case **00053332**) reports a **3rd-party vendor** getting **HTTP 403** when **programmatically referencing pages** (PDF disclosure URLs, e.g. `https://www.firstinterstatebank.com/disclosures/disclosure/128/pdf`). The customer assumed it was **IP-based** and asked Launch to **coordinate with Cloudflare to whitelist new vendor IPs** (e.g. `69.84.87.254`, `69.84.91.254`) while retaining the old ones.

Later detail: their engineering team's **cURL test still 403s**; they shared a **`cf-ray` ID** (`…-IAD`) and noted the client is **Java** with **User-Agent `Java/1.8.0_472-b08`**.

_Keywords: HTTP 403 programmatic access, whitelist IPs Cloudflare, cf-ray, Java user agent Java/1.8.0, WAF managed rules, Super Bot Fight Mode, Bot Management, Browser Integrity Check, skip WAF rules, no IP restrictions Launch, vendor cURL 403, bot blocked PDF URLs._

ANSWER

## Not an IP restriction — Cloudflare security (WAF / Bot / Browser Integrity Check)

- **Launch has no IP-based restrictions** — the site is accessible from any IP, so **IP allowlisting alone wasn't the fix.**
- After IP allowlisting was done and the **403 persisted** (confirmed via the customer's **cURL test + `cf-ray` ID**), the cause was identified as **Cloudflare security protections blocking the request** — **WAF (managed + custom rules), Rate Limiting, Super Bot Fight Mode, and ultimately Browser Integrity Check** — not IP rules.
- The **Java client / `User-Agent: Java/1.8.0_472-b08`** is a classic **bot-like signature** that **Bot Management / Browser Integrity Check** flags → 403 for automated/headless clients even though browsers work.

## What actually fixed it

1. **IP allowlisting** (done first) — necessary but **not sufficient.**
2. **Incomplete WAF skip rule** was the real gap: the existing **skip rule only bypassed Super Bot Fight Mode.** Updated it to **skip all WAF components for this traffic** — **custom rules, managed rules, rate limiting, AND Super Bot Fight Mode.**
3. **Final missing piece:** **Browser Integrity Check** was still blocking the traffic — updated that rule too. After that, the vendor's programmatic requests succeeded (no more 403).

## How to investigate (what support asked for)

- **How is the vendor accessing the pages?** (exact **cURL / script / method**) and the **client User-Agent**.
- The **`cf-ray` ID** of a failing request → lets the Cloudflare team **find the exact rule that blocked it** in the logs.
- Whether the project's **Edge Function** contains any **IP-restricting logic** (share the code) — rule that out too.

## Generalized guidance (for future similar queries)

- **"Programmatic/automated access gets 403, but browsers work"** → it's almost always **Cloudflare bot/WAF protections (Bot Management, Super Bot Fight Mode, Browser Integrity Check, managed/custom WAF, rate limiting)**, **not** an IP restriction. **Launch has no IP allowlist** of its own.
- **A bot-like `User-Agent`** (e.g. **`Java/...`**, generic HTTP libraries, headless tools) is a prime trigger. The durable fix is to **skip the relevant WAF/bot components for that traffic** — and remember **Browser Integrity Check is a separate toggle** from Bot Fight Mode (skipping bot rules alone may not be enough).
- **Always get the `cf-ray` ID** of a blocked request — it pinpoints the exact Cloudflare rule.
- IP allowlisting is fine to do but **won't resolve a bot/WAF block**; don't stop there.
- Related: [no fixed egress IPs](launch-no-fixed-outbound-egress-ip-allowlist.md), [Imperva WAF / Cloudflare Error 1000](launch-imperva-waf-cloudflare-error-1000-host-header.md), [branded reverse proxy — disable WAF-bot-JS challenges](../custom-domains-ssl/launch-branded-click-tracking-reverse-proxy-rewrites-requirements.md).

## Status

- **Resolved.** Cause: **Cloudflare security protections** (not IP). Fix progression: **IP allowlist → expand the WAF skip rule** (custom + managed + rate-limiting + Super Bot Fight Mode) **→ also skip Browser Integrity Check** (the final blocker). Vendor confirmed **programmatic access works, no more 403.**
