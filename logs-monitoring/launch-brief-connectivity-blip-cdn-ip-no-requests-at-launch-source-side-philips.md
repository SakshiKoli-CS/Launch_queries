QUESTION

Customer **Philips** (SF case **00046956**, Org `bltb414e71bd2e61150`, Project `ph-nextjs-web` `67a5d6832181bda09f767cb4`, EU) reported a **~5-minute connectivity issue** yesterday ~17:00 CET connecting **from their Akamai CDN to Contentstack Launch**. **Direct connectivity to the origin was fine**, and it **resumed on its own**. They gave the **Akamai source IP `88.221.24.231`** and a test `curl` with an `x-original-domain: stg.philips.ie` header, and asked Launch to **check whether traffic was blocked by the firewall**. Follow-up: they saw a **5-second timeout in one log** and asked whether any **Launch scaling/deployment** happened in that window.

_Keywords: brief connectivity issue Akamai CDN to Launch, 5 minutes intermittent, resumed on its own, source IP generated no requests at Launch, domain accessible from other Akamai IPs, no firewall block found, no request-timeout logs, x-original-domain header, scaling deployment in window, transient source-side network issue._

ANSWER

## Finding — the customer's specific CDN IP generated NO requests at Launch (issue was source-side)

- Investigating by the shared IP: **`88.221.24.231` generated NO requests at Launch during the reported window.** Meanwhile the **domain was functional and reachable from OTHER Akamai IPs** at the same time (e.g. `23.59.176.70`, `23.220.165.16` showed requests).
- **No firewall block** was found for that traffic, and **no request-timeout logs** at Launch for the incident.
- **Conclusion:** if requests from a given source IP **never arrived at Launch** while the same domain served fine from other IPs, the problem is **at the source / network path (that Akamai edge node or route)**, **not** on the Launch side. Asked the customer to **verify source-side network connectivity**.

## On the 5-second timeout / "did scaling or a deploy happen?"

- Reviewed: **no request-timed-out logs** at Launch for that window, and **no scaling/deployment event** correlated with the incident. A single 5s timeout observed in the customer's own logs did not correspond to any Launch-side timeout/scaling record.

## Generalized guidance (for future similar queries)

- **Brief (few-minute) CDN→Launch connectivity blip that self-resolves, origin fine** → check **whether the customer's source IP actually generated requests at Launch.** If the specific **source IP shows NO requests** while the **same domain served other IPs fine**, the fault is **source-side / network path**, not Launch.
- **Correlate against Launch signals:** look for **firewall blocks**, **request-timeout logs**, and **scaling/deployment events** in the window. **Absence of all three** + a self-healing blip + no-requests-from-that-IP strongly indicates a **transient upstream/CDN-node/routing issue on the customer side.**
- Provide the customer the **IPs that DID reach Launch** (proof the domain was reachable) so they can compare against their CDN node logs.
- Related: [downtime with no server logs = regional/upstream incident](launch-downtime-no-server-logs-azure-regional-incident.md), [intermittent pages not loading — regional (Australia)](launch-intermittent-pages-not-loading-regional-australia.md), [504 timeouts = customer backend, not Launch](launch-504-timeouts-customer-backend-not-launch-prod-points-to-dev-domain.md), [intermittent TCP ETIMEDOUT/ECONNREFUSED — CDA autoscaling spike](../networking-connectivity/launch-intermittent-tcp-etimedout-econnrefused-cda-autoscaling-spike.md).

## Status

- **Answered / closed — no Launch-side fault.** The reported **Akamai source IP `88.221.24.231` generated NO requests at Launch** during the window, while the **domain was reachable from other Akamai IPs** (`23.59.176.70`, `23.220.165.16`); **no firewall block** and **no request-timeout logs** were found, and **no scaling/deployment** correlated with the 5s timeout the customer saw. Pointed the customer to **verify source-side network connectivity** (transient upstream/CDN-node issue).
