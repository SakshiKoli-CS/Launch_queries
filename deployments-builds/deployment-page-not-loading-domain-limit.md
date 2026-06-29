QUESTION

In the **Welltower** org (uid `blt7499e10ceddc3b32`), the Launch UI **deployment page** for the production environment was not loading. Deployment pages for other environments in the same project were accessible. The affected project and environment:

- Project: `696588437f4d3e4591252587`
- Production env: `696588447f4d3e459125258f`

ANSWER

**Root cause — platform-side DB query limit**

The Launch service had a **default DB query limit of 30** when fetching domains for an environment. Welltower's production environment had **32 custom domains**, which exceeded this limit. The deployment page requires the **default domain URL** to render — because the query was truncated at 30 results, the default domain was not returned, causing the page to break.

Other environments with fewer domains were unaffected.

**Resolution**

Platform-side fix deployed to production. After the fix, both the production and UAT deployment pages loaded correctly.

**Suggested check for similar reports**

If a deployment page fails to load in the Launch UI while other environments in the same project work fine, check whether the affected environment has an unusually high number of custom domains. The domain count exceeding a service-level query limit is a known trigger for this failure mode.
