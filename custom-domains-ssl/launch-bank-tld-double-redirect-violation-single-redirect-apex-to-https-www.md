QUESTION

Customer **First Interstate Bank** (`firstinterstate.bank`, a **`.bank` TLD**) reports a **double-redirect** that **violates the `.bank` TLD registry regulations**: `http://firstinterstate.bank` → `http://www.firstinterstate.bank` (still HTTP, adds www) → `https://www.firstinterstate.bank/`. The `.bank` registry is strict about redirect handling. Can Launch fix it to avoid the double hop?

_Keywords: .bank TLD double redirect violation, apex http to www http to https, two 308 redirects, registry regulations strict, single redirect apex to https www, Varnish 308 Permanent Redirect, redirect chain reduce hops._

ANSWER

## Cause — a two-hop redirect chain (HTTP apex → HTTP www → HTTPS www)

- The apex was doing **two 308 redirects**: `http://firstinterstate.bank` → `http://www.firstinterstate.bank/` (adds www but **stays HTTP**) → then → `https://www.firstinterstate.bank/`.
- The **`.bank` TLD registry prohibits this double redirect** — it wants a **single, direct redirect to the final HTTPS destination.**

**Before fix** (`curl -IL firstinterstate.bank`):
```
308 → Location: http://www.firstinterstate.bank/     (hop 1, still HTTP)
308 → Location: https://www.firstinterstate.bank/    (hop 2)
```

## Fix — collapse to a single redirect straight to `https://www`

**After fix:**
```
308 Moved → Location: https://www.firstinterstate.bank/   (single hop)
```
- The apex now redirects **directly to `https://www.firstinterstate.bank/` in one step** — no intermediate HTTP-www hop → **compliant with the `.bank` TLD rules.**

## Generalized guidance (for future similar queries)

- **Multi-hop redirect chain (esp. `http-apex → http-www → https-www`)** → collapse to a **single redirect to the final `https://www` destination.** Strict TLDs like **`.bank`** treat double redirects as a **violation**; even outside those, fewer hops is better for performance/SEO.
- **Verify with `curl -IL <domain>`** — count the `Location:` hops; the goal is **one** redirect landing on the canonical **HTTPS** host.
- Related (same customer / apex cert): [apex cert renewal / move apex to Launch (Fastly redirect)](launch-apex-cert-renewal-expired-redirect-move-apex-to-launch.md), [apex→www redirect requires apex configured](launch-apex-to-www-redirect-requires-apex-configured-custom-hostnames.md), [redirect loop cached 1 year](../caching-cdn/launch-redirect-loop-308-cached-redirect-smaxage-1year-cache-control.md).

## Status

- **Resolved.** The apex was double-redirecting (`http` apex → `http` www → `https` www), **violating `.bank` TLD rules.** Fixed to a **single 308 straight to `https://www.firstinterstate.bank/`** (verified via `curl -IL`).
