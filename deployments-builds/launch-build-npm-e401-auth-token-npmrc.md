QUESTION

A Launch deployment (BMW demo, EU region, branch `pre-soc`) **failed during "Installing Dependencies"** with an **npm `E401` authentication error**, even though the project **builds fine locally** (`npm run build` succeeds, Next.js 14.2.3, all pages generated).

Build log (key lines):

```
—— Step 2: Installing Dependencies ——
npm error code E401
npm error Incorrect or missing password.
...
Installing dependencies failed
Deployment failed: Please try to redeploy the site, or contact support for assistance.
```

The user was confused: "Password to what? I have no password configured locally and it works." Redeploying and deploying a zip gave the same error.

_Keywords: npm E401, Incorrect or missing password, Installing dependencies failed, deployment failed npm, .npmrc, npm auth token, NPM_TOKEN, private npm registry, fontawesome npm, GitHub packages registry, works locally fails on Launch, build environment credentials._

ANSWER

## Root cause

The build failed because npm could **not authenticate to a private npm registry** during dependency install. The **"password" in the `E401` is the npm auth token, not a literal password.**

- It **works locally because the developer's npm credentials/tokens are cached on their machine** (e.g. in their user `~/.npmrc`).
- The **Launch build environment does not have those cached credentials**, so any dependency pulled from a private/authenticated registry fails with `E401`.

In this case the project's `.npmrc` referenced **multiple private registries with auth tokens that were not being supplied in the Launch build**:

```
@awesome.me:registry=https://npm.fontawesome.com/
@fortawesome:registry=https://npm.fontawesome.com/
//npm.fontawesome.com/:_authToken=xxxx
//npm.pkg.github.com/:_authToken=xxxx
```

(Font Awesome Pro registry + GitHub Packages registry.)

## Fix

- Ensure the **`.npmrc` auth tokens are available in the Launch build environment**, not just locally. The build succeeded once the project's required tokens were included.
- Practically: make the tokens available to the build (committed/templated `.npmrc` that reads tokens from env vars, and the corresponding **env variables set on the Launch project**), so the build can authenticate to each private registry.

## Things to check when you see npm `E401` on Launch (but local builds pass)

1. **`.npmrc` not picked up** in the build environment, or its `_authToken` lines reference tokens that aren't set there.
2. **Token expired or revoked** on npmjs / the private registry.
3. **Env vars not set / outdated on Launch** (e.g. `NPM_TOKEN`, `FONTAWESOME_TOKEN`, GitHub Packages token) — these exist on the dev's machine but must be configured on the Launch project.
4. **Token lacks permission** to access the private package/scope.
5. Multiple private registries — **every** `//registry/:_authToken` line in `.npmrc` needs a valid credential in the build env, not just the main one.

## Why "it works locally" is a red herring

Local success only proves the dependencies are reachable **with the developer's cached credentials**. The Launch build machine is a clean environment with no cached npm login — every private registry it hits must be authenticated explicitly via `.npmrc` + env vars.

## Resolution

- **Resolved.** The demo's `.npmrc` had Font Awesome and GitHub Packages registry tokens that weren't being included in the build; once those were supplied, the deployment succeeded.
