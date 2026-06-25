QUESTION

Customer **First Interstate Bank** (domain `www.firstinterstate.bank`, `.bank` gTLD via **FTLD**) needed to **update their TLS security certificate**. They supplied the new cert. Two complications arose:

1. The first key file was an **ENCRYPTED PRIVATE KEY** with no indication whether it was password-protected — couldn't be used as-is. Customer re-sent confirming it **was password protected** and provided the **password** (redacted).
2. After the `www` cert was updated, FTLD issued an **expired-cert notice**: the **`www` cert was correct, but the apex (barename) `firstinterstate.bank` redirect was still serving the old, expired cert**.

How is the apex configured differently from `www`, and how do we stop this recurring?

_Keywords: update security certificate, custom certificate renewal, expired cert apex redirect, .bank FTLD, encrypted private key password protected, apex domain not in Launch, apex to subdomain redirect Fastly, add apex domain in Launch, update A record, move apex away from Fastly._

ANSWER

## Private key must be usable — confirm encrypted vs password-protected

- The customer first sent an **encrypted private key** with no indication if it was **password protected**. We **can't tell from the file alone** — request either an **unencrypted private key** or the **decryption password**.
- Resolution: customer confirmed it **was password protected** and supplied the password (`<redacted>`); cert was then updated. **Redact cert body + password — never store either in the KB.**

## Root cause of the recurring "expired cert" — apex was NOT in Launch

- Only the **`www` subdomain was added in Launch**. The **apex/barename domain (`firstinterstate.bank`) was NOT in Launch** — its **apex→subdomain redirect was being handled by Fastly**, which kept serving the **old expired cert**.
- So updating the `www` cert in Launch **did not fix the apex** — different system (Fastly) owned that redirect and its (stale) cert.
- Immediate fix: corrected the apex so it **no longer shows the expired-cert error**.

## Prevention — add the apex domain in Launch (move it off Fastly)

- **Historically Launch did not support apex domains**, which is why apex→subdomain redirection was offloaded to **Fastly**. **Launch now supports apex domain addition**, so the recurrence-proof fix is to **bring the apex into Launch**:
  1. **Add the apex domain in Launch.**
  2. **Update the apex domain's A record** in DNS to point at the **Launch IP** (in this case from the **Fastly** IP `151.101.66.137` → the **Launch** IP `162.251.205.14`).
  3. Launch then **manages the apex's custom certificate** alongside `www` — **no more split ownership / stale cert**.
- **No downtime expected** during this change.

## Generalized guidance (for future similar queries)

- **"Cert updated but apex/barename still shows expired"** → check **what owns the apex redirect**. If `www` is in Launch but the **apex is handled externally (e.g. Fastly)**, updating the Launch cert **won't touch the apex** — the external layer keeps serving the old cert.
- **Fix the split:** **add the apex in Launch** + **repoint the apex A record** from the external IP to the **Launch IP**, then Launch manages both certs. Apex is now supported (it wasn't historically — that's why redirects were offloaded).
- **Encrypted private key supplied?** Don't guess — ask for an **unencrypted key or the password**; you can't tell from the file whether it's password protected. Redact the password and cert body in any record.

## Status

- **Resolved.** Updated `www` cert; fixed the **apex expired-cert** error; recommended **adding the apex in Launch + A-record repoint** (Fastly `151.101.66.137` → Launch `162.251.205.14`) so Launch owns both certs and the issue can't recur. Customer thanked the team.
