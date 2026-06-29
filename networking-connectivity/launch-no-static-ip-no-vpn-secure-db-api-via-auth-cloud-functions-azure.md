QUESTION

Prospect **Molteni** (via Deloitte Italy) evaluating Launch on **Azure EU**. They want to **securely connect Cloud Functions to their APIs / an Azure-hosted database** and ask:

- Are there **static IPs for Launch (Azure EU)** for allowlisting?
- How to **connect directly from Launch/Cloud Functions to a database on a hyperscaler (Azure)** — they expect **IP whitelisting** or **VPN** as the two recognized approaches.
- **Logging** recommendations (most-used by Launch customers)?
- **Repo/CI integration** — create a project from an **internal Git repo**? **Azure DevOps** native or via **repository mirroring**?

_Keywords: static IPs Launch Azure EU, IP whitelisting, VPN VPC peering not supported, secure database connection Cloud Functions, Azure-hosted database, dynamic outbound IPs autoscaled, auth API keys OAuth service tokens Azure API Management, Log Targets OpenTelemetry logging, internal git repo Azure DevOps repository mirroring Launch Deploy Pipeline CLI._

ANSWER

## 1. No static IPs — and no VPN/VPC peering; secure via AUTH, not network isolation

- **Launch does NOT provide a fixed set of static IPs** for allowlisting — the platform is **autoscaled / distributed**, so outbound IP ranges are **highly dynamic, change frequently, and are too numerous to maintain.** Static IPs **cannot be guaranteed.**
- **Private network connectivity (VPN / VPC peering) is NOT supported** (and **not planned in the near future**). Running within a customer VPC / network-layer isolation introduces significant **latency, scalability, and operational** trade-offs.
- **Recommended secure approach = application-layer authentication**, not IP/network controls:
  - **API keys, OAuth, service tokens**, or a **gateway like Azure API Management** in front of the APIs/DB.
  - For a **DB on Azure** that Cloud Functions connect to **directly** (supported — credentials via **environment variables**), secure it with **DB auth/credentials (+ TLS)** managed as env vars, fronted by a gateway/allowed-auth rather than IP allowlisting.
- **Bottom line to set expectations:** the customer's two assumed options (**IP whitelisting**, **VPN**) **aren't available on Launch**; **auth-based access is the supported, recommended pattern.**

## 2. Logging — Log Targets + OpenTelemetry

- Use **Log Targets** to **stream runtime and build logs to external systems.** Integrate with an **OpenTelemetry log collector** via Log Targets for enterprise-grade observability. (This is the common path for Launch customers needing external/centralized logging.)

## 3. Repo / CI integration

- **GitHub or Bitbucket** repos integrate with Launch **directly.**
- **Internal repos / Azure DevOps** (not natively connected) → use the **Launch Deploy Pipeline** approach: a **CI/CD pipeline (e.g. Azure DevOps) that pushes to Launch via the CLI**, or **repository mirroring** — enabling a fully automated deployment workflow.

## Generalized guidance (for future similar queries)

- **"Static IPs to allowlist Launch against our DB/API?"** → **No** — outbound IPs are **dynamic (autoscaled)**; **VPN/VPC peering also not supported / not planned.** **Secure with auth** (API keys / OAuth / service tokens / Azure API Management gateway), not IP or network isolation.
- **Cloud Functions can connect directly to a DB** (creds via **env vars**) — protect it with **DB credentials + TLS + a gateway/auth**, since you can't pin Launch's source IPs.
- **Logging:** **Log Targets → external system / OpenTelemetry collector.**
- **Repo/CI:** GitHub/Bitbucket connect directly; **Azure DevOps / internal Git → Launch Deploy Pipeline (CLI push) or repo mirroring.**
- Related: [no fixed outbound/egress IPs](launch-no-fixed-outbound-egress-ip-allowlist.md), [M2M CI/CD deploy via Launch API](../integrations/launch-m2m-cicd-deployment-api-app-region-host.md), [GitLab → Launch API + CI](../integrations/launch-gitlab-support-launch-api-ci-integration.md), [Log Targets OTEL TLS cert](../logs-monitoring/launch-log-targets-otel-tls-handshake-self-signed-cert-needs-ca-san.md).

## Status

- **Answered (presales).** **No static IPs** (dynamic autoscaled infra) and **no VPN/VPC peering** (not planned) → secure DB/API access via **auth (API keys/OAuth/service tokens/Azure API Management)**; Cloud Functions connect to a DB directly with **creds in env vars + TLS/gateway**. **Logging via Log Targets + OpenTelemetry.** **Repo/CI:** GitHub/Bitbucket direct; **Azure DevOps/internal → Deploy Pipeline (CLI) or mirroring.** Customer still preferred IP/VPN; reiterated those aren't supported and auth is the path.
