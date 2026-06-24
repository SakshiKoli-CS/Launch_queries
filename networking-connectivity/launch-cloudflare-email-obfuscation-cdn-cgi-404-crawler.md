QUESTION

Over the last few days, all mentions of **`support@contentstack.com`** started getting **flagged as 404s** across docs and `.com` (seen in crawler/SEO reports and Cloudflare logs). Was **email obfuscation enabled/added for Launch**, or is this a recent Launch change — or something else?

The reported 404 rows all point to the same URL **`/cdn-cgi/l/email-protection`** with anchor `[email protected]`.

_Keywords: email obfuscation, /cdn-cgi/l/email-protection 404, support@contentstack.com flagged 404, Cloudflare Scrape Shield, Email Address Obfuscation, crawler 404, Screaming Frog Ahrefs, SEO report false 404, not a Launch change._

ANSWER

## Cause — Cloudflare Email Address Obfuscation (Scrape Shield), not Launch

- This is **expected Cloudflare behavior**, **not a Launch change**. `contentstack.com` sits behind **Cloudflare**, whose **Scrape Shield → Email Address Obfuscation** feature rewrites any `mailto:` or plain-text email in the HTML (e.g. `support@contentstack.com`) into a placeholder link: **`/cdn-cgi/l/email-protection#<hex>`**, plus a small JS snippet that decodes it back in the browser.
- **Real users are unaffected:** humans see and click `support@contentstack.com` normally (the JS decodes it).
- **Crawlers** (Screaming Frog, Ahrefs, etc.) and any **direct hit** to the placeholder URL get a **404 by design** — `/cdn-cgi/l/email-protection` is **not a real page**.
- **Signature:** every flagged row pointing to the same `/cdn-cgi/l/email-protection` with anchor `[email protected]` is the classic Email Obfuscation fingerprint.

## Where it happens

- The transformation occurs **at the Cloudflare edge, after Launch serves the HTML.** **Launch does not touch email links** — so it's not a Launch feature toggle or regression.

## What to do

- **Nothing is broken** — these 404s are **benign and can be ignored** in crawl/SEO reports (exclude the `/cdn-cgi/l/email-protection` path).
- If the timing changed recently, it reflects a **Cloudflare-side** behavior change, not Launch.
- If you need to **stop specific emails from being obfuscated**, wrap them in `<!--email_off-->...<!--/email_off-->` (see the related entry for the CSP-impact case).

## Generalized guidance (for future similar queries)

- **"Emails / support@ showing as 404 in our crawl report"** → if the 404 URL is **`/cdn-cgi/l/email-protection`**, it's **Cloudflare Email Address Obfuscation** (Scrape Shield), expected and harmless for real users. **Not a Launch change.**
- **Ignore/exclude** these from SEO/broken-link reports; they are crawler-only artifacts, not real broken pages.
- To exclude specific addresses from obfuscation, use **`email_off`** comments.
- Related (same Cloudflare feature, different symptom — CSP blocking the decode script breaks real mailto links): [launch-cloudflare-email-protection-csp-nonce-mailto.md](launch-cloudflare-email-protection-csp-nonce-mailto.md).

## Status

- Confirmed to the reporter as **expected Cloudflare Email Obfuscation behavior**, not a Launch change; the 404s can be **ignored** in reports.
