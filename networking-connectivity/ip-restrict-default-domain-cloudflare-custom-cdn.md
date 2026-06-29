QUESTION

**MicroStrategy** (org `blt45a2a52367b47001`, AWS NA, SF **00056438**) requested that their Launch default domain URLs be **IP-restricted in Cloudflare** so that only traffic originating from their own CDN's IP ranges could access the default Launch environment URLs. They are running their own CDN on top of Launch and want to prevent the default domains from being publicly accessible directly.

Domains requested for restriction:
- `community.contentstackapps.com`
- `community-dev.contentstackapps.com`

ANSWER

**This is a platform-team action — not self-serve.**

IP-restricting a specific `*.contentstackapps.com` default domain to allowlisted IP ranges is done by the **Launch platform / DevOps team** in Cloudflare. There is no UI in the Launch dashboard for customers to configure inbound IP restrictions on default domains themselves.

**Use case**

When a customer places their own CDN (Akamai, Fastly, Cloudflare Enterprise, etc.) in front of Launch, they typically want:
1. All public traffic to enter via their CDN (for WAF, DDoS protection, edge logic)
2. The Launch default domain to be inaccessible directly from the public internet — so that bypassing their CDN is not possible

IP-restricting the default domain to the customer's CDN egress IPs achieves this. Only requests from those IPs reach the Launch origin; all other traffic is blocked at Cloudflare.

**Action taken**

Platform team (Vinesh) blocked public access to the two domains and restricted them to the customer's whitelisted IPs via Cloudflare. Confirmed working.

**How to request**

Route through the platform/DevOps team with:
- The specific `*.contentstackapps.com` domains to restrict
- The customer's CDN egress IP ranges to allowlist
- Org UID and region for context

**Related**

- For customers asking about Launch's own outbound/egress IPs: [No fixed outbound egress IPs for allowlisting](launch-no-fixed-outbound-egress-ip-allowlist.md)
- For multi-CDN cache invalidation considerations: @caching-cdn/launch-cloudflare-apo-cache-purge-multi-cdn-invalidation.md
