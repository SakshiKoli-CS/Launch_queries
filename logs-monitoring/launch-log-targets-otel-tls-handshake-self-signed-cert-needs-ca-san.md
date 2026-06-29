QUESTION

Customer **The Jackson Laboratory** (SF case **00052370**) is forwarding Launch logs into **Splunk** via an **OTEL gRPC endpoint** using **Launch Log Targets**. Logs aren't arriving. Setup:

- Log target points to a **GCP server** running an **OTEL collector**, port **4317**, with a **self-signed TLS certificate**; bearer token configured.
- Splunk support verified the OTEL collector config is correct.
- **Packet captures show the client (Contentstack) resetting the connection** during the TLS handshake. With TLS disabled on the server, dumps show the **client attempting TLS but the server saying it doesn't support TLS.**

_Keywords: Log Targets OTEL gRPC Splunk, logs not reaching, port 4317, self-signed certificate rejected, TLS handshake failed, x509 certificate relies on legacy Common Name use SANs, Subject Alternative Name missing, CA-signed certificate Let's Encrypt, connection reset client, authentication handshake failed._

ANSWER

## Cause — the OTEL endpoint's TLS certificate is rejected (self-signed + legacy CN, no SAN)

Contentstack-side error during the handshake:
```
rpc error: code = Unavailable desc = connection error: desc = "transport: authentication handshake failed:
tls: failed to verify certificate: x509: certificate relies on legacy Common Name field, use SANs instead"
```

Two problems with the OTEL collector's certificate:

1. **Self-signed cert is not trusted.** Launch Log Targets require a **CA-signed certificate** so the client can establish trust during the TLS handshake. **Self-signed certs are rejected.**
2. **Cert relies on the legacy Common Name (CN) field, with no SAN.** Modern TLS/gRPC clients **no longer validate CN** — they require the server's **DNS name or IP in the Subject Alternative Name (SAN)** field. Missing SAN → verification fails.

## Fix

- **Replace the self-signed cert with a CA-signed TLS certificate** (e.g. via **Let's Encrypt**).
- **Ensure the cert includes the correct DNS name or IP in the SAN extension.**
- Once a **CA-signed cert with SAN** is in place, the **TLS handshake succeeds and logs flow.**

## Generalized guidance (for future similar queries)

- **Log Targets → OTEL/gRPC endpoint, logs not arriving, client resets the connection mid-handshake** → it's a **TLS trust failure on the customer's endpoint**, not a Launch bug. Two classic causes:
  - **Self-signed certificate** → use a **CA-signed** cert (Let's Encrypt etc.); Launch won't trust self-signed.
  - **`x509: certificate relies on legacy Common Name field, use SANs instead`** → the cert has **no SAN**; add the server's **DNS name / IP to the SAN** extension (CN alone is no longer accepted).
- Diagnostic tell: **packet capture shows the Contentstack client initiating TLS then resetting** = handshake/verification rejected at the client.
- Reference for a working collector setup: **`contentstack-launch-examples/launch-cloudwatch-otel-collector`** (note: it's an **AWS CloudWatch** example — adapt for other clouds like GCP).
- Related: [server logs 5k UI limit → Log Targets](launch-server-logs-ui-5000-entry-limit-use-log-targets.md).

## Status

- **Resolved.** Root cause: OTEL collector used a **self-signed cert relying on legacy CN with no SAN**, so Launch's TLS handshake failed (`x509 … use SANs instead`). Fix: **CA-signed certificate (e.g. Let's Encrypt) with the DNS/IP in the SAN.** Customer fixed the OTEL collector and is now **receiving Contentstack logs.**
