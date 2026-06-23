QUESTION

Customer **SRPNet** (Org ID `blt6d328981cb625d90`, Azure NA) hit a connectivity issue integrating the **Form.io Marketplace App**. Their Form.io environment is **behind a firewall**, only reachable on their internal network, and credential validation from the Marketplace App returned **403 Forbidden**.

Contentstack confirmed the **Marketplace App outbound IPs** (`20.98.104.159`, `20.115.208.50`) and shared them so the customer could allowlist them on Form.io. The customer then asked: **can the same outbound IPs be allowlisted for Contentstack Launch-hosted applications (DEV/UAT/PROD)?** Their Next.js app on Launch (Project `69e0842b51a2c6d702b08cbc`) also needs to reach the firewalled Form.io environment, so they wanted **fixed Launch egress IPs to allowlist.**

_Keywords: Launch outbound IP, egress IP, static IP allowlist, fixed IP, firewall, 403 Forbidden, Form.io, Marketplace App outbound IPs, allowlisting Launch, IP allowlisting not supported, private/internal endpoint, proxy gateway static IP._

ANSWER

## Core answer — Launch does NOT provide fixed outbound/egress IPs

- The **Marketplace App** and **Launch-hosted applications** run on **separate infrastructure**. Allowlisting the **Marketplace App** outbound IPs only fixes connectivity **for the Marketplace App** — it does **not** apply to Launch-hosted apps.
- **Launch does not provide fixed/static outbound IP addresses.** Therefore **IP-based allowlisting is not a supported approach** for enabling connectivity from a Launch-hosted app to a firewalled endpoint like Form.io.

## Recommended approaches (instead of IP allowlisting)

To let a Launch-hosted Next.js app reach a firewalled service:

1. **Expose the endpoint publicly and secure it with authentication** — API keys, OAuth, JWT, etc. (rather than relying on network-level IP restriction).
2. **Use a customer-managed proxy/gateway with a static public IP** that *can* be allowlisted on the Form.io side; the Launch app talks to the proxy, the proxy (with its fixed IP) talks to Form.io.

## Roadmap

- **Static / egress IP support on Launch is on the roadmap** — but not available today.

## Resolution

- Customer **implemented API-Key–based authentication** (option 1) for the Form.io endpoint and the **case was closed.**

## Generalized guidance (for future similar queries)

- **"What are the outbound/egress IPs for my Launch app so I can allowlist them?"** → Launch has **no fixed outbound IPs**; IP allowlisting is **not supported** for Launch-hosted apps. (Static egress IP is roadmap, not available.)
- **Don't conflate Marketplace App IPs with Launch** — they're separate infrastructure. Marketplace App outbound IPs only cover Marketplace App connectivity.
- **To reach a firewalled/private service from Launch**, secure the endpoint with **auth (API key/OAuth/JWT)** and expose it, **or** route through a **customer-managed proxy/gateway that has a static IP** to allowlist.
- A **403 Forbidden** during credential validation from a firewalled service is typically network/IP allowlisting, not an app bug — confirm whether the caller's source IP is allowlisted (and remember Launch's isn't fixed).
