QUESTION

**Milestone Systems A/S** (SF **00054206**; Launch project `68d29b7ceaa2ca1bb35ac4d0`, prod env `691617a8644cd489d7e5d373`) reported that end users at one corporate customer could not access their Launch-hosted Customer Portal via the custom domain `customer.milestonesys.com`. Their corporate DNS filtering application (Windows Defender) was blocking access.

The blockage was linked to the Launch default domain `milestone-customer-portal.eu-azcontentstackapps.com`, which triggered "Suspected Phishing" warnings and was being flagged by **5 engines on VirusTotal**. The application did not appear to be making explicit calls to the default domain — the customer was unsure how end users were reaching it. The site worked correctly when the corporate DNS filter was disabled. The customer was concerned that other end users might be silently affected.

ANSWER

**What's happening**

The Launch default domain (`*.eu-azcontentstackapps.com`) was flagged by security scanning engines (VirusTotal: 5 engines; Windows Defender). Corporate DNS filters and endpoint-security tools subscribe to these threat-intel feeds, so once the default domain appears on a blocklist, **any request that touches that domain — even indirectly — can trigger a block**, even if the user is accessing the site via a custom domain.

This is a **different mechanism from the Google Safe Browsing / browser-level flag** documented in [launch-contentstackapps-dangerous-site-malware-canonical-rca-fix.md](launch-contentstackapps-dangerous-site-malware-canonical-rca-fix.md), which affected `*.contentstackapps.com` (AWS NA). Here the flagged domain is the **Azure EU** default (`eu-azcontentstackapps.com`) and the blocking layer is a **corporate DNS filter**, not the browser.

**Why the default domain gets involved even with a custom domain configured**

Even when a custom domain is the primary entry point, the default Launch domain may still be referenced via:
- Hard-coded URLs in application code, environment variables, or config files pointing to the default domain
- Browser redirects or OAuth callbacks that resolve through the default domain
- Third-party tools (uptime monitors, crawlers, SEMRush) hitting the default domain directly

Confirming that no such references exist is the first debugging step.

**Investigation findings**

- Accessing `http://milestone-customer-portal.eu-azcontentstackapps.com` directly showed a **"Suspected Phishing"** warning
- VirusTotal report flagged the domain across 5 engines: [VirusTotal report](https://www.virustotal.com/gui/url/9a4bc4e94cc81808e47eb3510aec105f2210a4cfe6f689f928376fb5026d3a14/detection)
- Custom domain `customer.milestonesys.com` was not independently flagged

**Workarounds**

- **Immediate (IT-side):** The corporate IT team can allowlist `*.eu-azcontentstackapps.com` or the specific default subdomain in their DNS filter / Windows Defender policy
- **App-side:** Audit the codebase for any references to the default Launch domain and replace with the custom domain. Check environment variables, config, OAuth redirect URIs, and any server-side fetch calls

**Security checks to avoid re-flagging**

Security scanners flag domains that exhibit phishing-like patterns. Verify the following are clean to reduce the likelihood of future flags:

- HTTPS enforced (no HTTP fallback)
- No open redirects (query-param or header-injection-based)
- No suspicious redirect chains involving external domains
- No embedded third-party scripts from untrusted or unrecognised domains
- Login page does not visually mimic Microsoft, Google, or other brand UIs

**Resolution status**

Open — customer following up. No confirmation received that the issue is resolved. The allowlist workaround and security checklist have been shared; awaiting customer response to close.

**Related entries**

- [Google Safe Browsing flag on `*.contentstackapps.com` (AWS NA) — canonical](launch-contentstackapps-dangerous-site-malware-canonical-rca-fix.md)
- [Multi-customer malware flag + RCA](launch-default-domain-malware-flag-multi-customer-rca.md)
