QUESTION

A user reported that **some pages on contentstack.com were not launching / the site appeared down** (intermittently). Notably, this was the **5th report in a few weeks from users in Australia** — each time the site **appeared down and then started working again** on its own.

_Keywords: intermittent site down, pages not loading, site appears down then works, region-specific Australia, intermittent availability, transient outage, hard to reproduce, Observe Cloudflare monitoring._

ANSWER

## Nature of the issue

- **Intermittent and not consistently reproducible** — the site appeared down, then recovered without action. The recurring signal was that reports clustered from **Australia** over a few weeks, suggesting a **regional/edge** component rather than a whole-site outage.

## Investigation approach (for intermittent reports)

Because it couldn't be reproduced on demand, the key was collecting **occurrence details** from the reporter:
- **Time** the issue occurred (with timezone),
- **Error/status code** seen,
- **Pages/URLs** affected,
- Whether any **recent code/config changes** were made.

Platform-side, behavior was reviewed via **Observe** and **Cloudflare** monitoring around the reported windows.

## Resolution

- The issue was **identified and a fix implemented** platform-side, then **monitored** (via Observe + Cloudflare) with no recurrence observed afterward.
- _(Specific root cause was not detailed in the thread; treat as a platform-side intermittent/edge issue that was remediated. If a similar regional intermittent pattern recurs, escalate with the occurrence details above.)_

## Generalized guidance (for future similar queries)

- **"Site/pages intermittently down then recover, hard to reproduce"** → gather **time + timezone, status code, affected URLs, recent changes**; correlate with **Observe** and **Cloudflare** monitoring for the exact windows.
- **A geographic cluster (e.g. repeated reports from one country/region)** points to a **regional/edge or upstream-network** factor, not a full-site outage — note the pattern explicitly when escalating.
- Recovery-on-its-own + no app changes = likely **platform/edge transient**, not customer code.

## Status

- **Identified and fixed** platform-side; monitored via Observe + Cloudflare with no further impact. Asked the reporter to confirm resolution on their end.
