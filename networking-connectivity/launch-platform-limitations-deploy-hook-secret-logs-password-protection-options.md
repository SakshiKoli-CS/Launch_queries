QUESTION

Customer **Eight25Media** (SF case **00058579**) raised platform feedback / limitations spanning three areas:

1. **Deploy Hooks have no secret/auth:** no support for a secret token or custom auth header on the deploy-hook endpoint — anyone with the URL could trigger a deployment.
2. **Server log viewer usability:** can't **pause** auto-scrolling; intermittent **"No logs"** (even while logs are generating); **high memory usage / browser-tab crashes** during long sessions.
3. **Password protection breaks CORS preflight `OPTIONS`:** with password protection enabled, browser **preflight `OPTIONS`** requests are challenged/blocked, so authenticated cross-origin API calls fail. Is this expected? Workaround? Plans to exempt OPTIONS?

_Keywords: deploy hook secret, deploy hook authentication, secure webhook token, server logs pause, no logs intermittent, log viewer memory crash, password protection OPTIONS blocked, CORS preflight 401, basic auth bypass OPTIONS, Edge Functions custom password protection, Launch Public API deploy, Automate Secure HTTP Trigger._

ANSWER

## 1. Deploy Hooks — no native secret (by design); the URL *is* the secret

- Launch Deploy Hooks are **intentionally simple, unique URL-based trigger endpoints** with **no native secret validation or auth header** on the hook URL. The **uniqueness/opacity of the URL is the implicit access control** — treat it as a secret.
- This is a **deliberate design choice**, consistent across the hosting ecosystem (e.g. **Vercel Deploy Hooks** work the same way — unique URL, no signature/token on the hook itself). Keep the URL confidential, store it as a **protected secret in CI/CD** (GitHub Actions secrets, GitLab CI variables), and **rotate if compromised**.
- For **authenticated, auditable, programmatic** deployment control, use **Launch Public APIs** (full token-based auth) — the right tool, **not a workaround**.
- Optional extra layer: **Automate Secure HTTP Triggers** (validate a configured secret before triggering). Filed as a **feature request** for native hook secrets.

## 2. Server log viewer — known limitations, on the backlog

- The native viewer offers **real-time streaming with no pause/freeze**. The **no-pause**, intermittent **"No logs"**, and **memory/crash** issues during long sessions are **acknowledged as valid** and added to the product backlog — tracked under an **Epic (CL-4123)**.
- **Clarification on Log Targets:** forwarding logs to an **external observability platform via Log Targets** is recommended for **long-running analysis, retention, filtering, search** — but it was **not** offered as a *substitute* for fixing the native viewer; those are separate concerns.

## 3. Password protection + CORS preflight OPTIONS — by design; use Edge Functions

- Launch password protection is an **infrastructure-level, environment-wide gate** that blocks **all** traffic **before it reaches the application/origin** — intended to lock down staging/preview environments entirely. It is **coarse-grained** and **not request-aware** (no understanding of CORS, preflight, or route-level exemptions).
- Therefore **every request — including `OPTIONS` preflight — is challenged at the edge.** This is a **deliberate architectural boundary**, not a config oversight: selectively exempting methods/routes requires **application-level routing awareness**, which the infra protection layer can't assume.
- **Intended mechanism (not a workaround): Edge Functions.** Use an Edge Function to **pass `OPTIONS` through while protecting other routes** (custom, request-aware password protection). This is the same pattern other platforms use (e.g. Vercel Middleware for custom auth/CORS).
  - Ref: Edge Functions docs + **"Launch Edge custom Password Protection" example** (contentstack-launch-examples repo).

## Reframing — Edge Functions / Public APIs are the *intended* layer

- Launch is a **lightweight, composable hosting platform** (SSR/CSR, edge compute, cache, deployment lifecycle). **Application-layer behaviors** — "exempt OPTIONS from auth", "apply auth to specific routes", "verify a webhook secret" — are expressed via **Edge Functions / Public APIs / Automate**, by design, rather than as toggles in a protection UI. The platform provides the compute primitive; the request-aware logic is the app's to define.

## Generalized guidance (for future similar queries)

- **"Deploy hook has no secret/auth — security risk?"** → By design (URL = secret, like Vercel); store it as a CI/CD secret, rotate if leaked. For authenticated/auditable triggers use **Launch Public APIs**; optionally gate via **Automate Secure HTTP Trigger**.
- **"Can't pause server logs / 'No logs' flicker / browser crashes on long log sessions"** → Known native-viewer limitations (Epic **CL-4123**); use **Log Targets** for heavy/long-running analysis (separate from the viewer fixes).
- **"Password protection breaks my CORS preflight / authenticated cross-origin calls"** → Expected: password protection challenges **all** requests (incl. `OPTIONS`) at the **infra edge** before the app. To exempt `OPTIONS`/specific routes, implement **custom password protection via an Edge Function** (the intended, request-aware mechanism).
- **General principle:** request-aware/app-level behavior on Launch belongs in **Edge Functions** (or Public APIs/Automate), not in infra toggles — this is intentional, not a gap.

## Status

- Detailed clarifications + design rationale shared; deploy-hook secret raised as a **feature request**; server-log improvements tracked under **Epic CL-4123**; OPTIONS exemption addressed via the **Edge Function custom password-protection** pattern. Customer acknowledged.
