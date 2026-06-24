QUESTION

Customer **Ello Group** (SF case **00058576**) reported the **server logs would not load** for their **production** environment (tastecard). Clearing cache didn't help. Symptoms: a **403 error in the console** and a **503 on screen**; **nothing in the network tab**. Stack key `bltd44a95fddc9e64be` (EU).

Follow-up after the fix: when a **new chunk of logs loads, the viewer auto-scrolls to the bottom**, making it very hard to read earlier logs — scrolling up gets interrupted every ~5–10s as new logs arrive and the view jumps back down.

_Keywords: server logs not loading, 403 console 503 screen, logs won't load, cache clear no help, log viewer broken, CL-4060, auto-scroll to bottom, can't scroll up logs, log viewer autoscroll, real-time log streaming, Log Targets._

ANSWER

## Part 1 — Server logs not loading (403/503) → platform bug, fixed

- The logs-not-loading failure (**403** in console / **503** on screen, empty network tab) was a confirmed **Launch platform bug**, tracked as **CL-4060**.
- The team identified it, made changes with cross-environment testing, and pushed the fix to **production**. Resolved.

## Part 2 — Log viewer auto-scrolls to bottom (separate UX issue)

- The native log viewer **streams logs in real time**, and on each new batch it **auto-scrolls to the latest entries** — so scrolling up to review historical logs gets interrupted every ~5–10s. This is **reproducible** and **acknowledged**, shared with the product team for a log-viewer-experience improvement (no pause/freeze/anchor today).
- This is a **separate UI concern** from the 403/503 loading bug — related to the broader server-log-viewer usability backlog (see [platform limitations entry](../networking-connectivity/launch-platform-limitations-deploy-hook-secret-logs-password-protection-options.md), Epic CL-4123).
- **Workaround for extended analysis:** configure **Log Targets** to forward logs to an external observability platform (better search/filtering/retention than the native viewer).

## Access note (recurring detour)

- Investigating required project access; two snags came up (same as other cases):
  1. **SSO tenant block** — the support engineer's account wasn't an external user in the customer's Microsoft tenant; SSO login failed until the customer **enabled login without SSO** (or added the account to the tenant).
  2. **Stack vs Launch project invite** — the initial invite was to the **Stack**, not the **Launch project**; need an invite to the **Launch project** specifically (with admin access) to investigate Launch issues.

## Generalized guidance (for future similar queries)

- **"Server logs won't load (403/503), cache clear doesn't help, empty network tab"** → likely a **platform-side log-viewer bug** (cf. CL-4060), not customer config; check for an active fix/incident before deep-diving the project.
- **"Log viewer keeps auto-scrolling to the bottom / can't read older logs"** → known native-viewer limitation (no pause/anchor; CL-4123 backlog). For extended/historical analysis use **Log Targets** → external observability.
- **Granting support access:** ensure the account is added to the customer's **SSO tenant** (or SSO temporarily disabled) **and** invited to the **Launch project** (not just the Stack).

## Status

- 403/503 loading bug **fixed in production** (CL-4060). Auto-scroll behavior **acknowledged**, with product reviewing log-viewer improvements; Log Targets offered for heavy analysis.
