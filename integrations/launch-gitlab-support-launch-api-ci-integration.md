QUESTION

Does **Launch support GitLab** (as a Git provider, like it does GitHub/Bitbucket/Azure Repos)?

_Keywords: GitLab support, GitLab CI/CD, no native GitLab, Launch APIs GitLab pipeline, trigger deployments from GitLab, launch-api-gitlab-ci-example, integrate GitLab Launch._

ANSWER

## No native GitLab support — but you can integrate via Launch APIs

- **GitLab is not natively supported** as a connected Git provider in Launch.
- Customers **can still use GitLab** by calling the **Launch APIs from their GitLab CI/CD pipeline** to **trigger and manage deployments** as part of their CI/CD workflow.

## Reference implementation

- Use the official example repo, which shows a **sample GitLab pipeline config** and how to **authenticate and call the Launch APIs** to trigger/manage deployments:
  - **`contentstack-launch-examples/launch-api-gitlab-ci-example`** — https://github.com/contentstack-launch-examples/launch-api-gitlab-ci-example

## Generalized guidance (for future similar queries)

- **"Do we support GitLab in Launch?"** → **Not natively.** Point the customer to the **Launch APIs + GitLab CI integration** path and the **`launch-api-gitlab-ci-example`** reference repo.
- The same pattern (drive deployments via Launch APIs from an external CI) applies to **any unsupported Git provider / custom CI** — authenticate, then call the deployment APIs from the pipeline.

## Status

- Answered: **no native GitLab support**; integrate via **Launch APIs in GitLab CI**, with the reference example repo provided. Customer satisfied.
