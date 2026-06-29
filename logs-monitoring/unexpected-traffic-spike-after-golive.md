QUESTION

Shortly after the Welltower Monarch site went live, the Welltower team observed a noticeable spike in request volume on the Monarch site. The spike was not present on the Brandycare site during the same timeframe. The customer asked whether the spike was expected following a go-live switch and whether it could be bot traffic.

ANSWER

**Context / When this pattern appears**

Traffic spikes reported shortly after a site go-live or DNS switch are a recurring concern. Customers typically notice them when:

- A new site starts receiving real production traffic for the first time
- Two sites under the same account behave differently in traffic volume
- Requests originate from data center IPs rather than residential/browser IPs, triggering bot suspicion

In most cases, these spikes are **not caused by the switch itself** and are **not bot traffic** — they reflect application-side behavior (analytics calls, logging, metrics pings) that only becomes visible once the site is live and receiving users.

**Investigation**

Cloudflare HTTP traffic reports for both sites were reviewed. Key differences:

- **Monarch**: elevated request volume, predominantly to `/api/log` and `/api/log/metrics`; traffic origin is US-based data center IPs
- **Brandycare**: no comparable spike; no similar logging or metrics endpoint traffic observed

The spike was isolated to specific API endpoints rather than distributed across pages or static assets — a strong signal that this is application-generated, not a broad bot sweep.

**How to distinguish bot traffic from app-triggered traffic**

| Signal | Bot traffic | App-triggered traffic |
|---|---|---|
| Endpoints hit | Varied, often probing (`/admin`, `/.env`, random paths) | Specific, predictable (`/api/log`, `/api/metrics`) |
| Response codes | Many 4xx (probing non-existent paths) | Mostly 2xx or cached responses |
| Correlation with app code | None | Matches known instrumentation in the codebase |
| User agent | Scrapers, headless browsers, known bot UAs | Server-side / no UA (internal call) |
| IP origin | Rotating IPs, many countries | Consistent data center IPs tied to server-side execution |
| Volume pattern | Steady or random | Scales with real user visits |

**Cause**

The `/api/log/metrics` requests are **application-triggered**: they fire whenever a user visits the Monarch site (server-side or edge-side call from within the app). Because Brandycare does not implement the same logging or metrics instrumentation, no equivalent traffic appears there. The data-center IP origin is expected for server-side outbound calls — it is not a sign of external probing.

This pattern is not a side-effect of the go-live switch itself. It reflects a **difference in application implementation** between the two sites that only becomes visible once Monarch is live and under real user load.

**Resolution**

No platform-side action required. The site was responding normally throughout:

- No elevated 5xx errors
- No downtime or performance degradation
- Requests returning successful or cached responses

The behavior is application-specific and not a security or platform concern.

**Suggested checks**

- Confirm with the dev team whether the logging/metrics calls are intentional and firing at the expected rate
- If volume seems disproportionate to actual user visits, check for duplicate triggers (e.g., metrics fired multiple times per page load, missing deduplication guards)
- If the customer wants to reduce noise in Cloudflare reports, they can filter by endpoint or set up a separate analytics view scoped to non-API paths
