QUESTION

A customer's Jenkins pipeline failed during a Launch CLI deployment with:

```
TypeError: withDisabledDeprecations is not a function
```

The error occurred during the deployment logs polling step from `@contentstack/cli-launch`. Setup details:

- CLI install: `npm install @contentstack/cli --save-dev --before=2026-03-17`
- Node.js: v22.15.0 (Jenkins)
- Command: `npx csdx launch -c <launch-config> --redeploy-latest`
- Project type: FileUpload (Next.js), AZURE-EU (SF **00056576**, JIRA **CL-3336**)
- Last successful deployment: March 18; no Node.js or dependency changes made

The `--before` date flag was added by the customer's security team to prevent installing potentially compromised package versions (SHA1-Hulud npm supply chain concern).

ANSWER

**Root cause — `--before` flag interaction with `@contentstack/cli@1.59.0`**

Using `npm install @contentstack/cli --save-dev --before=2026-03-19` resolves to `@contentstack/cli@1.59.0`. That version, when installed via the `--before` flag, exhibits the `withDisabledDeprecations is not a function` error during the polling step.

Notably, installing the **same version explicitly** without the flag:

```bash
npm install @contentstack/cli@1.59.0 --save-dev
```

does **not** reproduce the error. The bug is triggered specifically by the `--before` flag installation path, not by the version itself or by Node.js v22/v24 compatibility.

Upgrading to Node.js v24 did **not** resolve the issue — Node version is not the cause.

**Workaround — install the version explicitly**

Replace:
```bash
npm install @contentstack/cli --save-dev --before=2026-03-19
```
With:
```bash
npm install @contentstack/cli@1.59.0 --save-dev
```

This maintains the pinned version the security team requires while avoiding the `--before` flag behavior that triggers the error.

**Platform fix**

Root cause identified and fixed platform-side (JIRA **CL-3336**), deployed to production. Customer confirmed deployment works after applying the workaround.

**Suggested checks for similar reports**

If `withDisabledDeprecations is not a function` appears in CLI deployment logs:
1. Check whether `--before=<date>` is used in the npm install command
2. Replace with explicit version pinning (`@x.y.z`) — same security outcome, avoids the flag behavior
3. Confirm Node.js version is not a factor (error reproduced on both v22 and v24)
