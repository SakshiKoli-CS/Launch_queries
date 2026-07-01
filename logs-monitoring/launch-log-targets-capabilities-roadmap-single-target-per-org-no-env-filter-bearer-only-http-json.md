QUESTION

Customer **Philips / EPAM** (security-conscious, high-visibility global brand) asks about the **current capabilities and future roadmap of Launch Log Targets**: how many Log Targets can be configured, whether logs can be **filtered by environment**, what **authentication/header formats** are supported, and whether other **ingestion models** (temporary log storage with API, log agents like Fluent Bit/Logstash, generic HTTP/JSON) are planned.

_Keywords: Log Targets number per organization single target, filter logs by environment not supported workaround enrich identifiers, authentication Authorization Bearer token only OTLP gRPC, alternative auth header format New Relic, temporary log storage API access not planned, log agents Fluent Bit Logstash Datadog roadmap, generic HTTP JSON ingestion alternative protocols roadmap Q4, log targets feature roadmap._

ANSWER

## Current capabilities (as of this thread)

- **Number of Log Targets:** **one (single) Log Target per organization.**
- **Environment filtering:** **not supported** natively — no built-in way to filter logs by environment. **Workaround:** **enrich logs in the application with environment-specific identifiers**, then filter **downstream** in your log-processing/visualization tool. *(Note: Launch later began adding **resource attributes — Project/Environment/Deployment ID** — to Log Target requests; see the OTLP/gRPC setup entry for the server-side filtering path.)*
- **Auth / header format:** **only `Authorization: Bearer <token>`** for **OTLP/gRPC**. No alternative auth mechanisms yet (alternatives are planned but **not near-term**) — a point of friction for integrations like **New Relic** that want different header/auth approaches.

## Roadmap / not-planned (uncommitted)

- **Generic HTTP / JSON ingestion (alternative protocols):** **on the roadmap**, planned for upcoming quarters — **the most likely next item.** No firm timeframe; **soonest start ~Q4**, subject to holistic quarter planning. Customer interest **directly helps prioritize** it.
- **Log-agent integrations (Fluent Bit, Logstash, Datadog, etc.):** **being considered** for common agents; may explore if demand is common.
- **Alternative authentication mechanisms:** planned but **not in the near future.**
- **Temporary log storage with API access:** **NOT planned.**

## Generalized guidance (for future similar queries)

- **"How many Log Targets can we have?"** → **one per organization.**
- **"Can we filter logs by environment?"** → historically **no** (enrich logs with env identifiers app-side and filter downstream); Launch has since started attaching **resource attributes (Project/Environment/Deployment ID)** — check the current setup entry for whether server-side env filtering is available.
- **"What auth does Log Targets support?"** → **`Authorization: Bearer <token>` only**, over **OTLP/gRPC**. Alternative auth/header schemes are **future, not near-term** (relevant for New-Relic-style integrations).
- **Roadmap framing (uncommitted):** **generic HTTP/JSON ingestion** is the most likely upcoming item (earliest ~Q4, no commitment); **log agents** under consideration; **more auth methods** further out; **temporary log storage + API** not planned. **Customer demand explicitly feeds prioritization** — capture it.
- Related: [Log Target OTLP/gRPC + Bearer Token + Datadog setup (+ resource attributes for env filtering)](launch-log-targets-otlp-grpc-endpoint-bearer-token-datadog-resource-attributes-project-env-filter-pods.md), [log visibility → forward via Log Targets (OTel/gRPC/Collector), roadmap items](launch-log-visibility-retention-ui-access-rights-forward-via-log-targets-otel-grpc-collector-philips.md), [log targets OTel TLS handshake / self-signed cert needs CA+SAN](launch-log-targets-otel-tls-handshake-self-signed-cert-needs-ca-san.md), [server-logs UI 5,000-entry limit → use Log Targets](launch-server-logs-ui-5000-entry-limit-use-log-targets.md).

## Status

- **Answered (capabilities + roadmap).** Current: **1 Log Target/org**, **no native env filtering** (workaround: app-side env identifiers → downstream filter; later augmented by **resource attributes**), **auth = Bearer token only over OTLP/gRPC**. Roadmap (uncommitted): **generic HTTP/JSON ingestion** = most likely next (earliest ~Q4), **log agents** under consideration, **more auth methods** further out, **temporary log storage + API** not planned. Team to **align internally on Q4 priorities**; customer interest in HTTP/JSON helps drive it.
