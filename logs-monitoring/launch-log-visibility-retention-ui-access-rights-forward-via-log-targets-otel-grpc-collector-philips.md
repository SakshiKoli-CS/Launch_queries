QUESTION

Customer **Philips** raised several **log-visibility** questions about Launch: **log retention & limits**, **log UI usability**, and **access rights** (who can view deployments/logs). They were **offered a temporary intermediate OTLP exporter service** (based on the `launch-cloudwatch-otel-collector` example repo) but were **not keen on running the repo for now**. What can Launch offer on the raised points?

_Keywords: log retention limits Launch, log UI usability, access rights view logs deployments, read-only collaborator, forward server logs to aggregator, Log Targets OTel gRPC, OpenTelemetry export standard, target does not support OTel or gRPC deploy own OTel Collector, launch-cloudwatch-otel-collector repo, HTTP support log targets roadmap, bearer token auth only, marketplace apps log aggregators._

ANSWER

## Access rights → add them as a Read-only collaborator

- To let the customer **view deployments and logs without write access**, add them as a **Read-only collaborator** in Launch. This is the answer to the "access rights" ask.

## Log retention / limits / UI usability → use a log aggregator via Log Targets

- The **Launch server-logs UI is intentionally basic** (recent logs, limited retention/search — see related on the 5,000-entry UI limit). Advanced **retention, search, filtering, analysis** are **not built into the UI**; the intended path is to **forward all server logs to the customer's own log-aggregation platform.**
- **Launch exports logs via Log Targets** to the customer's preferred aggregator:
  - Logs are forwarded in **OpenTelemetry (OTel)** format — the widely-adopted open standard.
  - Transport is **gRPC** — chosen for performance / near-real-time delivery.
- **Confirm the customer's aggregator supports OTel ingestion over gRPC.** If it does, point Log Targets at it directly.
- **If the aggregator does NOT support OTel/gRPC**, the customer can **deploy their own OTel Collector** in their infrastructure; **Launch forwards logs to that Collector**, which **relays to their chosen platform** (translating protocol/format as needed). Reference: **`launch-cloudwatch-otel-collector`** example repo. (Launch can offer to help stand up an intermediate collector, but the customer runs it.)

## Roadmap items mentioned (not yet prioritized)

- **HTTP support for Log Targets** (in addition to gRPC).
- **More authentication methods** for Log Targets (today: **bearer token** only).
- **Marketplace apps** for specific aggregators that don't support OTel.
- General log-UI improvements are "taken into consideration" but not committed.

## Generalized guidance (for future similar queries)

- **"Can we get better log retention / search / UI in Launch?"** → the **UI is deliberately minimal**; **forward logs to your own aggregator via Log Targets** for retention/search/analysis.
- **Log Targets = OTel over gRPC.** Confirm the aggregator supports **OTel ingestion over gRPC**. If not, run an **OTel Collector** on the customer side (Launch → Collector → their platform) — see the `launch-cloudwatch-otel-collector` example.
- **"Who can see our logs/deployments?"** → add them as a **Read-only collaborator** (view-only, no write).
- Roadmap (uncommitted): **HTTP Log Targets, more auth methods beyond bearer token, marketplace apps for non-OTel aggregators.**
- Related: [server-logs UI 5,000-entry limit → use Log Targets](launch-server-logs-ui-5000-entry-limit-use-log-targets.md), [log target Datadog setup](log-target-datadog-setup.md), [log targets OTel TLS handshake / self-signed cert needs CA+SAN](launch-log-targets-otel-tls-handshake-self-signed-cert-needs-ca-san.md), [non-prod log volume estimate](launch-nonprod-log-volume-estimate-log-targets.md), [server logs not loading 403/503](launch-server-logs-not-loading-403-503-autoscroll.md).

## Status

- **Answered / guided.** **Access rights** → add the customer as a **Read-only collaborator** (view deployments + logs, no write). **Retention/search/UI** → not in the UI by design; **forward logs to their aggregator via Log Targets (OTel over gRPC)**; if the aggregator lacks OTel/gRPC, **run an OTel Collector** on their side (Launch → Collector → platform; `launch-cloudwatch-otel-collector` example). Customer **declined running the example repo for now**. Noted roadmap items (**HTTP Log Targets, more auth methods, marketplace apps**) as **not yet prioritized**.
