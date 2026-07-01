QUESTION

Customer **PODS** is setting up a **Launch Log Target → OTel Collector → Datadog** pipeline and has detailed questions: **how to configure the OTLP/gRPC endpoint**, **how bearer-token auth works**, **what log format** Launch sends, **how to test the connection**, and — after logs started flowing — **why they can't distinguish staging vs production logs** in Datadog.

_Keywords: Log Target OTLP gRPC endpoint config, TLS-secured gRPC port 443 to 4317, azurecontainerapps.io collector URL, bearer token auth bearertokenauth extension Authorization header, log format OTLP Protobuf OpenTelemetry, grpcurl test invalid token rejected, debug exporter Datadog dashboard server edge logs, no test-log support test-grpc-logs.sh, cannot distinguish staging vs production logs, resource attributes Project ID Environment ID Deployment ID filter, HTTP log targets roadmap._

ANSWER

## 1. OTLP/gRPC endpoint — must be TLS-secured; proxy 443 → 4317

- Launch requires a **TLS-secured OTLP/gRPC endpoint.** On the collector, **proxy HTTPS/TLS traffic from port 443 → port 4317** (gRPC receiver) — either on a specific path or the root path.
- The **Log Target endpoint URL** then points at **`:443`**, e.g.:
  ```
  https://contentstack-otel-collector--0000001.<hash>.eastus.azurecontainerapps.io:443
  ```
- Once the 443→4317 TLS proxy is in place, **log ingestion starts working.**

## 2. Authentication — a Bearer Token you generate in the Log Target (no API key/service account)

- In the **Launch Console → Log Target setup**, **generate and set a Bearer Token.** Use that **same token** in the OTel Collector's **`bearertokenauth`** extension.
- **All requests from Launch include your token in the `Authorization` header as a Bearer Token** → the collector authorizes matching requests and rejects the rest.
- **No service account or API key needed** — the Bearer Token is the only credential.

## 3. Log format — OTLP / Protobuf

- Launch sends logs in **OTLP (OpenTelemetry Protocol) / Protobuf** format, **directly to the OTel Collector**.
- **No special headers/metadata required** — defaults are sufficient.

## 4. Testing the connection

- Launch **does not currently support sending test logs from the Console** (feature under consideration, along with **HTTP-based Log Targets**).
- To test from your side:
  - Use **`grpcurl`** to send a request with the **correct** Bearer Token (should be **allowed**) and one with an **invalid** token (should be **rejected**) — validates auth.
  - Use the OTel Collector **debug exporter** to inspect received logs, and/or check the **Datadog dashboard** for ingested logs.
  - Use **`test-grpc-logs.sh`** from the example repo to send test logs yourself.
- When flowing correctly, you should see **both Server and Edge logs** in Datadog for **all Launch environments**.

## Distinguishing staging vs production (and per-project/deployment) — resource attributes

- **Known gap:** initially the forwarded logs **lacked attributes to tell staging vs production apart** in Datadog.
- **Fix (rolled out by Launch):** Launch **adds resource attributes to Log Target requests** so you can **filter by Project ID, Environment ID, and Deployment ID** — cleanly separating logs across environments/projects/deployments in Datadog.

## Generalized guidance (for future similar queries)

- **Log Target = TLS OTLP/gRPC.** Collector must expose a **TLS endpoint proxying 443 → 4317**; the Log Target URL uses **`:443`**.
- **Auth = a Bearer Token generated in the Log Target**, mirrored into the collector's **`bearertokenauth`** extension (sent as `Authorization: Bearer …`). No API key/service account.
- **Format = OTLP/Protobuf**, default headers. **No Console test-log feature yet** → test with **`grpcurl`** (valid vs invalid token), the collector **debug exporter**, and **`test-grpc-logs.sh`**.
- **Can't tell environments apart in the aggregator?** → Launch attaches **resource attributes (Project ID / Environment ID / Deployment ID)**; **filter on those** in Datadog. Both **Server and Edge** logs appear for all envs.
- Roadmap (uncommitted): **HTTP Log Targets**, **Console test logs**.
- Related: [log target Datadog setup](log-target-datadog-setup.md), [log targets OTel TLS handshake / self-signed cert needs CA+SAN](launch-log-targets-otel-tls-handshake-self-signed-cert-needs-ca-san.md), [log visibility → forward via Log Targets (OTel/gRPC/Collector)](launch-log-visibility-retention-ui-access-rights-forward-via-log-targets-otel-grpc-collector-philips.md), [server-logs UI 5,000-entry limit → use Log Targets](launch-server-logs-ui-5000-entry-limit-use-log-targets.md), [non-prod log volume estimate](launch-nonprod-log-volume-estimate-log-targets.md).

## Status

- **Resolved / guided.** Endpoint = **TLS OTLP/gRPC, proxy 443→4317**, URL on `:443`. Auth = **Bearer Token** from the Log Target, mirrored in the collector's **`bearertokenauth`** extension (no API key). Format = **OTLP/Protobuf**. Testing = **`grpcurl` (valid/invalid token) + debug exporter + `test-grpc-logs.sh`** (no Console test-log feature yet). **Staging-vs-prod distinction** was missing → Launch **added resource attributes (Project/Environment/Deployment ID)** so logs are **filterable** in Datadog (both Server + Edge logs for all envs).
