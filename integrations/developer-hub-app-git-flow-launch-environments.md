QUESTION

**ASICS** (via Arda Akbel) wanted to establish a **dev → staging → production gating system (git flow)** for their Developer Hub apps hosted on Launch. Most apps were in MRT; they were exploring whether Launch's environment model could better support this workflow. A follow-up question also arose from the same pattern with Airbnb.

The specific ask: can a single Launch project with multiple environments (dev/staging/prod) be wired up to Developer Hub so each environment's URL is used for the corresponding app stage?

ANSWER

**Current workaround — Custom Hosting with separate app configs per environment**

Launch's environment model supports this use case, but with a manual step on the Developer Hub side:

1. Create a **single Launch project** with separate environments for dev, staging, and prod
2. In **Developer Hub**, create a **separate app configuration** for each environment (dev app, staging app, prod app)
3. For each app, use the **Custom Hosting** option and manually enter the URL of the corresponding Launch environment

This provides full environment separation and aligns with a git-based promotion flow (develop branch → dev env, main branch → prod env, etc.).

**Limitations**

| Limitation | Detail |
|---|---|
| "Host with Launch" dropdown | Basic — lists projects but does **not** let you select a specific environment's URL. Use **Custom Hosting** (manual URL entry) instead |
| Multiple URLs per app | **Not supported** in Developer Hub. A single app cannot have different URLs per environment. Separate app configurations are required for non-production environments |
| Switching environments | Must manually update the Custom Hosting URL in Developer Hub when promoting between environments |

**Feature request (not yet available)**

The ideal solution — a Developer Hub option to select which Launch environment's URL to use directly — has been raised as a feature request. The "Host with Launch" dropdown needs enhancement to support environment-level selection.

**Recommended setup for ≥ 3 environments**

```
Launch Project
├── dev   environment  → Custom Hosting URL → Developer Hub "Dev App"   config
├── stage environment  → Custom Hosting URL → Developer Hub "Stage App" config
└── prod  environment  → Custom Hosting URL → Developer Hub "Prod App"  config
```

One Launch project is sufficient. Separate Developer Hub app configs per non-prod environment are needed until multi-URL support is added.

**Resolution**

Workaround communicated; customer (ASICS) to proceed with Custom Hosting approach. Feature enhancement for environment selection in Developer Hub is pending.
