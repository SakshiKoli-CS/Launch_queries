QUESTION

Customer **Five Iron Golf** (via Horizontal Digital, Org `blt560b79437bc41ae4`, SF case **00050890**) wants to **flip their canonical domain**: make **`fiveirongolf.com` (apex) the primary**, with **`www.fiveirongolf.com` redirecting to the apex** (currently it's the reverse — `www` is primary). Triggering context:

- During the **Cloudflare outage**, a **temporary 302 redirect** was placed; when removed, it left a **downstream issue where `https://fiveirongolf.com` stopped redirecting/loading** even with a **308** in place. The IT lead found the **apex A record had to be reset to the correct IP**.
- Questions: can the site load for **both apex and www**? Can they **revert to apex as primary**, and what's the lead time?

_Keywords: change primary domain www to apex, canonical flip, apex primary www redirect, 308 redirect apex not loading, A record reset correct IP, Cloudflare outage temporary 302 redirect downstream issue, domain entitlement increase prerequisite, add apex domain in UI, apex redirect resilient recovery, serve both apex and www._

ANSWER

## What's possible — Launch can serve apex + www in any redirect direction

- Launch can **serve traffic from both apex and www**, and configure **any** of:
  - **Apex serving the site directly**
  - **Apex redirecting to www**
  - **www redirecting to apex** ← what this customer wants
  - **No redirect** (both served independently)
- So **making `fiveirongolf.com` the primary with `www` → apex is supported.**

## Prerequisite — a domain entitlement increase

- Changing to this setup **required increasing their domain entitlement (by 1)** — you need the apex added alongside www, which can exceed the current domain limit. The reconfiguration **can't be completed until the entitlement is raised.**
- After the increase, the customer can **add their apex domain in the Launch UI** themselves, then the redirect is configured (often on a brief call).

## The outage-recovery lesson — why apex+redirect is the resilient setup

- The original breakage came from a **temporary redirect (Cloudflare outage) being removed**, after which the **apex A record was pointing wrong** → apex didn't load even with a 308.
- **Recommended approach:** configure the **apex domain with a redirect to the chosen primary subdomain** (Launch's general recommendation is apex→www, but the same resilience applies to the customer's chosen direction). Benefits: **consistent routing, simpler DNS, clearer control during temporary rerouting.**
- **Why it's more resilient:** if the apex points to Launch with a redirect, during a temporary reroute **only the apex A record changes**; **restoring the apex A record to Launch immediately brings the site back** because **subdomain routing keeps working untouched** (Launch manages its routing/SSL/CDN). No www-side changes needed → predictable recovery.

## Generalized guidance (for future similar queries)

- **"Make the apex (or www) the primary domain, with the other redirecting"** → **supported in any direction** (apex-direct / apex→www / www→apex / both). Confirm the **exact direction** with the customer (this thread had repeated www↔apex confusion).
- **Changing the setup may need a domain entitlement increase** — adding the apex alongside www can exceed the domain limit; raise it first, then the customer **adds the apex in the Launch UI** and the redirect is configured.
- **Resilient pattern:** apex points to Launch **with a redirect to the primary subdomain** → during a temporary reroute only the **apex A record** moves, and **restoring it instantly recovers** the site (subdomain routing/SSL/CDN unaffected). A removed temporary redirect + stale apex A record is a classic cause of "apex won't load even with a 308."
- Related: [apex→www redirect requires apex configured first](launch-apex-to-www-redirect-requires-apex-configured-custom-hostnames.md), [canonical apex→www (already working)](launch-canonical-apex-to-www-redirect-seo-already-working.md), [apex cert / move apex to Launch (Fastly redirect)](launch-apex-cert-renewal-expired-redirect-move-apex-to-launch.md), [add domains / domain-limit increase](launch-add-new-domains-decommission-repurpose-environment.md).

## Status

- **Done / in progress.** Flipping to **apex-primary with `www`→apex is supported**; required a **domain entitlement increase (+1)**, which was completed → customer could then **add the apex in the Launch UI** (done on a call) with next steps explained for their DNS team. Root of the original breakage: a **temporary outage redirect removed + stale apex A record** (reset to correct IP). Reinforced the **apex-with-redirect** setup for resilient recovery.
