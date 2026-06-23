QUESTION

Partner **Verndale** is setting up **Imperva as a WAF in front of a Launch** instance. Two things:

1. They want a redirect from **`https://nellhotels.com` → `https://www.nellhotels.com`** (apex → www).
2. **Imperva support told them to whitelist Imperva's IPs in the "webserver firewall"**, but they **can't find anywhere in Launch to do that** — and they don't have any **IP restrictions or password protection** configured in Launch. Is whitelisting a necessary step? Could this be related to how the domains are configured in Launch?

Going live today; this is the final hurdle.

_Keywords: Imperva WAF, whitelist Imperva IPs, webserver firewall, no IP restriction in Launch, apex to www redirect, nellhotels.com, IP-based access control Edge Function, custom domain redirect, Edge redirects._

ANSWER

## 1. Apex → www redirect

- Configure the `nellhotels.com` → `www.nellhotels.com` redirect via **Custom Domain** settings or **Edge redirects**. (Apex → subdomain is the natively supported redirect direction.)

## 2. Whitelisting Imperva's IPs in Launch

- **Launch has no "webserver firewall" UI to whitelist IPs.** There's no place to add Imperva's IPs because Launch doesn't expose a built-in IP-allowlist setting of that kind.
- **It isn't a necessary step in this case** — since the customer has **no IP restrictions or password protection configured in Launch**, **nothing on the Launch side is blocking Imperva's traffic.** Whitelisting would only matter if Launch were enforcing IP restrictions, which it isn't here.
- If they **do** want **IP-based access control** (e.g. to only allow Imperva and block direct origin access), that's implemented in code via an **Edge Function** — not a platform setting. Ref: **IP-Based Access Control Using Edge Functions.**

## Generalized guidance (for future similar queries)

- **"Where do I whitelist my WAF's IPs in Launch's firewall?"** → There's **no built-in IP-allowlist UI**. If no IP restriction/password protection is configured, **nothing is blocking the WAF** and whitelisting isn't needed. To *enforce* IP rules (e.g. allow only the WAF), use an **Edge Function** for IP-based access control.
- **Apex → www (or apex → subdomain) redirect** → supported via **Custom Domain** or **Edge redirects**. (For subdomain → subdomain, use an Edge Function — see the subdomain-redirect entry.)
- **Imperva/external WAF in front of Launch** → also ensure the WAF **forwards the correct `Host` header** to the Launch origin (the common failure mode — see the Imperva Error 1000 entry: [launch-imperva-waf-cloudflare-error-1000-host-header.md](launch-imperva-waf-cloudflare-error-1000-host-header.md)).
