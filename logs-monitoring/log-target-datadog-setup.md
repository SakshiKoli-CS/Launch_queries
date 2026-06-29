QUESTION

A customer (SF **00056123**) was forwarding Contentstack Launch logs to an **OpenTelemetry (OTEL) Collector** on Azure Container Apps and then exporting to **Datadog**. Two problems:

1. Logs contained **no environment identifiers** — all environments appeared identical in Datadog with no way to filter or group by environment (Production, Stage, Testing). Regex-based detection in OTEL config was unreliable due to inconsistent log patterns.
2. After switching to Launch's **HTTP Log Target** (direct Datadog integration), **logs were not appearing** in Datadog at all.

ANSWER

**Problem 1 — No environment metadata in OTEL-forwarded logs**

The OTEL Collector approach does not include environment identifiers in the log payload. Launch does not inject env metadata at that layer.

**Recommended approach: use Launch's HTTP Log Target directly**, bypassing the intermediate GRPC OTEL Collector. With this setup, logs include the following metadata in the headers:

- `environment_uid`
- `project_uid`
- `organization_uid`
- `deployment_uid`

The `environment_uid` allows reliable filtering and grouping by environment in Datadog without additional OTEL processing.

Reference: [Log Targets documentation](https://www.contentstack.com/docs/developers/launch/log-targets)

**Problem 2 — Logs not appearing in Datadog after switching to HTTP Log Target**

**Root cause: format mismatch between Launch's log format and the Datadog endpoint.**

Launch emits logs in **OTLP (OpenTelemetry Protocol) format**. The standard Datadog HTTP intake endpoint (`https://http-intake.logs.datadoghq.com/api/v2/logs`) expects **Datadog JSON format** — sending OTLP logs to this endpoint results in logs not appearing or being misrouted.

**Fix: use the Datadog OTLP endpoint instead.**

| | Endpoint | When to use |
|---|---|---|
| HTTP intake (JSON) | `https://http-intake.logs.us5.datadoghq.com/api/v2/logs` | Datadog JSON format logs |
| **OTLP (recommended for Launch)** | `https://otlp.us3.datadoghq.com/v1/logs` | OTLP format logs — matches Launch output |

> Note: Datadog endpoints are region-specific (`us3`, `us5`, etc.). Use the endpoint that matches your Datadog region.

**Required headers** (for either endpoint):

```
DD-API-KEY: <your_api_key>
Content-Type: application/json
```

**Filtering logs by environment in Datadog**

Once logs are flowing via the OTLP endpoint, filter by environment using the `service` field:

```
service: launch-<your-envUID>
```

To map raw `environment_uid` and `deployment_uid` values to human-readable names, create a **Datadog pipeline** (Logs → Pipelines) that remaps these fields to friendly labels.

**Resolution**

Customer confirmed logs are successfully received in Datadog via the OTLP endpoint and were able to create a pipeline mapping service/deployment IDs to custom friendly names.
