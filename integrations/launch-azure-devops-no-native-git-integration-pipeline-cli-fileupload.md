QUESTION

Customer deploys to Launch using **GitHub** and asks: **can they use Azure DevOps instead?** (A coworker's AI-assisted draft claimed Azure DevOps "can be used with some considerations" / "being considered for future development" — is that accurate?)

_Keywords: Azure DevOps with Launch, no native Git integration Azure DevOps, use pipeline to deploy, Launch CLI FileUpload artifact, supported git providers GitHub Bitbucket, launch-deploy-pipeline example, CI build then deploy to Launch, mirror repo._

ANSWER

## No native Azure DevOps Git integration — but deploy via pipeline + CLI

- **Direct Git integration for Azure DevOps is NOT available** in Launch (only **GitHub / Bitbucket Cloud** connect as native Git providers).
- **But you can still use Azure DevOps** by having it **build the site and deliver the artifact to Launch** via a pipeline:
  - **File Upload (via CLI):** run the **Launch CLI inside the Azure DevOps pipeline** to **upload the build artifact** to Launch.
  - Or **link the project to a supported Git provider** (GitHub / Bitbucket Cloud) instead.
- In this model, **Azure DevOps does the build and initiates the deploy**; **Launch receives the final build artifact / deploy trigger** — **no direct repository integration required.**
- Example: **`contentstack-launch-examples/launch-deploy-pipeline`** (CI/CD via the FileUpload method).

## Generalized guidance (for future similar queries)

- **"Can we use Azure DevOps with Launch?"** → **Not as a native Git provider** (GitHub/Bitbucket Cloud only). Use **Azure DevOps Pipelines + the Launch CLI (FileUpload)** to build and push the artifact to Launch, or connect a supported provider. Reference: **`launch-deploy-pipeline`** example.
- Same **"external CI → Launch via API/CLI"** pattern covers **any unsupported Git provider / custom CI** (GitLab, Azure DevOps, internal Git).
- (Don't over-promise on AI-suggested "coming soon" — native Azure DevOps Git integration is **not currently available**; the supported path today is pipeline + CLI.)
- Related: [GitLab → Launch API + CI](launch-gitlab-support-launch-api-ci-integration.md), [Cradlepoint GitLab/API POC](launch-cradlepoint-gitlab-api-deployment-poc-reference-implementation.md), [M2M CI/CD deploy (Azure DevOps) via Launch API](launch-m2m-cicd-deployment-api-app-region-host.md), [no static IPs → Deploy Pipeline / repo mirroring](../networking-connectivity/launch-no-static-ip-no-vpn-secure-db-api-via-auth-cloud-functions-azure.md).

## Status

- **Answered.** **No native Azure DevOps Git integration** (GitHub/Bitbucket Cloud only). Deploy by running the **Launch CLI (FileUpload)** inside an **Azure DevOps pipeline** to push the build artifact, or connect a supported provider. Example: **`launch-deploy-pipeline`**.
