QUESTION

Customer **UVA Health** (via WillowTree; Org `bltdcc012133cba1c83`, Stack `blt630bf358c48da3c9`, Project `6800171649412f0653c0eb70`, Azure NA, JIRA **CL-2094**) — **all pages** of `uvahealth.com` and `childrens.uvahealth.com` suddenly returned **403 Forbidden** (~9:30 AM EST, Nov 1). The site had been **live and fine for over a week** with no relevant change.

Confusing signals (initially framed as a redirect/edge-function problem):
- **cURL returned 200**; intermittent in browsers; **homepage `/` had no applicable redirect.**
- They had a `[proxy].edge.js` + a 4000+-redirect `launch.json`, so everyone suspected the **redirect system**.
- **Key discovery:** the site **loads when the `invoca_session` cookie is removed**, and **403s when it's present.** Further: **`invoca_session` was the only cookie in JSON format** — and **adding any new JSON-formatted cookie reproduced the 403.**

_Keywords: 403 Forbidden all pages, JSON formatted cookie blocked, invoca_session cookie, cURL 200 browser 403, WAF OWASP rule blocks JSON cookie, Azure not AWS, platform-wide WAF geo-blocking compliance change, extra OWASP rules inadvertently added, redirect edge function red herring, delayed impact week later, CL-2094._

ANSWER

## Root cause — a platform-wide WAF change blocked requests carrying JSON-formatted cookies (Azure)

- The 403 was **NOT** the customer's redirects/edge function (the red herring). It was a **Contentstack-wide WAF rules configuration change** (compliance **geo-blocking** for sanctioned countries) during which **extra rules — among the top OWASP rules — were inadvertently added.**
- One of those OWASP rules **blocked requests containing a JSON-formatted cookie.** The customer's **`invoca_session` cookie (set via Invoca through GTM) was JSON-formatted** → every request carrying it was **403'd at the edge.** Confirmed: **removing the cookie → site loads; adding any JSON-format cookie → 403 returns.**
- **Environment-specific:** the JSON-cookie 403 reproduced on **Launch sites hosted on Azure**, but **worked on AWS** — a tell that it was an **edge/WAF-layer difference, not the app.**

## Why the "worked for a week, then broke" + "cURL 200" confusion

- It "worked for a week" because the **403 began when the WAF change rolled out** — the **Invoca/JSON cookie was pre-existing**; the *rule* was new (delayed-looking impact, but it was the rule's deploy time, not the cookie).
- **cURL 200 vs browser 403** → cURL didn't send the offending JSON cookie; real browsers did. **A cached 403 / cookie presence** explained the intermittency.

## Fix

- The **inadvertently-added extra WAF (OWASP) rules were removed** — JSON-formatted cookies no longer 403. **Only the GEO-location WAF rule remains.**
- Interim customer workaround (before the platform fix): they **disabled the Invoca GTM script / cleared the `invoca_session` cookie.**
- Follow-ups: **24x7 monitoring added** for these sites; possible **reversal of the GEO-location WAF policy** later; **formal RCA** issued.

## Generalized guidance (for future similar queries)

- **Sudden 403 on all/most pages, cURL 200 but browser 403, "worked for a week," and it clears when a specific cookie is removed** → suspect a **WAF rule blocking by request content**, not the app/redirects. **Test by removing cookies** — here the trigger was **any JSON-formatted cookie** (an OWASP rule), not Invoca specifically.
- **Launch WAF is platform-wide** — a **compliance/geo-blocking WAF change** can inadvertently add OWASP rules that block legitimate traffic network-wide; **escalate to check recent WAF/OWASP changes.**
- **Azure-vs-AWS difference** in the same symptom points to an **edge/WAF layer**, not application code.
- **Don't anchor on the customer's framing** (here: "it's our redirect system / `[proxy].edge.js`") — the redirect config was a **red herring**; the cookie/WAF was the cause.
- Related: [403 Access Denied — same platform-wide WAF/OWASP change (PODS, path-based)](launch-403-access-denied-platform-wide-waf-rule-change-extra-owasp-rules.md), [programmatic 403 → Cloudflare WAF/Bot](launch-403-programmatic-access-cloudflare-waf-bot-browser-integrity-check.md), [Launch WAF model (no per-user rules)](../edge-functions-rewrites/launch-all-urls-redirect-to-homepage-nextjs-middleware-accept-header-waf-model.md).

## Status

- **Resolved (CL-2094).** Root cause: a **platform-wide compliance WAF/geo-blocking change** that **inadvertently added OWASP rules**, one of which **blocked requests with a JSON-formatted cookie** (the customer's `invoca_session`), causing site-wide **403s on Azure** (AWS unaffected). The redirect/edge-function framing was a **red herring.** **Extra rules removed** (GEO rule kept); **24x7 monitoring added**; formal RCA issued. This was a **major Launch-delivery incident** (also impacted Moncler with 403s; Bibby with a Googlebot-crawl variant).
