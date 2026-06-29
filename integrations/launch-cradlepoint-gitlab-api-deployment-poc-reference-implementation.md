QUESTION

Customer **Cradlepoint** has **deployment needs** and is weighing **using the Launch API vs a built-in GitLab integration**. Internal ask: coordinate a POC/reference they can be walked through, and set up a customer call.

_Keywords: Cradlepoint, GitLab integration vs Launch API, deploy via Launch API, no built-in GitLab integration, POC reference implementation, launch-api-gitlab-ci-example, GitLab CI deploy walkthrough._

ANSWER

## Use the Launch API + GitLab CI (no built-in GitLab integration)

- There is **no built-in GitLab integration**; the supported path is **calling the Launch API from GitLab CI** to trigger/manage deployments.
- A **reference implementation / POC** was created demonstrating exactly this:
  - **`contentstack-launch-examples/launch-api-gitlab-ci-example`** — https://github.com/contentstack-launch-examples/launch-api-gitlab-ci-example
- It's a **shareable example** to highlight Launch API capabilities for customers; a **guided walkthrough / call** can be offered alongside it.

## Generalized guidance (for future similar queries)

- **"GitLab integration vs Launch API for deployments?"** → **no native GitLab integration**; **drive deployments via the Launch API from GitLab CI**, using the **`launch-api-gitlab-ci-example`** repo as the reference/POC. Offer a walkthrough call for onboarding.
- This is the **same answer** as the general "does Launch support GitLab" question — see [GitLab support → Launch API + CI](launch-gitlab-support-launch-api-ci-integration.md). The reusable POC repo is the go-to artifact to share proactively.
- Same API-driven pattern applies to **any unsupported Git provider / custom CI**.

## Status

- **Answered / enablement.** Pointed to the **Launch API + GitLab CI** approach with the **`launch-api-gitlab-ci-example`** POC repo; offered a **walkthrough call** for the customer.
