QUESTION

Customer **UVA Health System** (via WillowTree/Telus; Org `bltdcc012133cba1c83`, Stack `blt630bf358c48da3c9`, Project `6800171649412f0653c0eb70`, `uvahealth.com`, SF case **00053033**, JIRA **CL-2863 / CLOUD-16476**) reports a **cluster of Launch problems** eroding their confidence:

1. **Frequent deploy failures** — both **immediately after "Cloning Repository"** and **at "Deploying Edge Functions."**
2. **"Successful" deployments that silently failed to build/deploy some content types** — 3 content types (Healthy Balance, Healthy Practice, Making of Medicine) didn't build, site showed **LIVE with big content gaps.**
3. **Odd deployment logs that appear to start mid-process.**

_Keywords: frequent deploy failures, fail after cloning repository, fail at deploying edge functions, successful deploy missing content types, site live with content gaps, logs start midway 5000 line limit, GitHub OAuth token expired connection issue, change project owner UI not possible backend transfer, electron install socket hang up node_modules, SSG full rebuild network failures, switch to SSR cache revalidation priming Astro, CDA didn't return all entries 422, Azure network hiccup._

ANSWER

This thread surfaced **several distinct issues** — keep them separate.

## (a) Odd logs "starting mid-deployment" — only the last 5,000 log lines are shown

- Deployment logs display **only the most recent ~5,000 lines.** Earlier steps **did run successfully**; their logs just aren't visible (long deploys exceed 5k lines). Not a real failure. (See also the [server-logs 5k limit](../logs-monitoring/launch-server-logs-ui-5000-entry-limit-use-log-targets.md).)

## (b) Deploy failures right after "Cloning Repository" — GitHub connection / OAuth token

- A batch of "Cloning repository… → Deployment failed: please redeploy or contact support" failures traced to a **GitHub connection / OAuth token issue** (token invalid/expired). Recovery: **disconnect GitHub from Launch → remove the GitHub app (github.com/settings/apps) → reconnect → fresh deploy → Repair project connection** (Launch UI → project → Settings → General → Git Connection). (Phrase to the customer as a **"GitHub connection issue."**)
- "Deploying Edge Functions" failures usually originate **application-side** — confirm recent edge-function code changes (customer had none recent).

## (c) Project-owner change — not in the UI, done from the backend

- **Launch has no UI to change the project owner.** Support can **transfer ownership from the backend** — provide the **new owner's email + Org UID + Project UID.** (Note: a disconnected GitHub repo may be **un-reconnectable until ownership is transferred** if the owner is an external/non-org user.)

## (d) Install failure: `electron` pulling a large binary → `socket hang up`

- A deploy failed at **Installing dependencies** with `npm error … node_modules/electron … RequestError: socket hang up` (during `node install.js`).
- **Electron downloads a large binary on every install**, raising the chance of **transient network failures.** Recommendations: **remove Electron from production builds if not needed**, or use a **prebuilt/cached Electron binary** to avoid the per-build download. Repeated reproduction (deploy every 15 min) **couldn't reproduce it** → consistent with a **temporary network glitch**; Cloud team investigating **Azure network hiccups** infra-wide.

## (e) "Successful deploy but missing content" / "build failed because CDA didn't return all entries"

- The customer clarified they **intentionally fail the build when the CMS doesn't return all expected entries** — the real question is **why CDA sometimes doesn't return all content.**
- CDA-side checks in the reported windows found **no 5xx and no errors**; however CDA logs did show **1000+ `422`s** from the customer's own calls: **wrong/deleted `entry_uid`** (e.g. `/content_types/header/entries/header`, deleted entry UIDs) and a **token without access to the branch** (`insufficient permissions … branch 'uvacms_894'`). These are **request/permission errors on the customer side**, investigated as a possible contributor; the "missing entries at build" remained under joint Launch+CDA investigation.

## Root architectural cause + recommendation (the durable takeaway)

- The **SSG approach (full site rebuild on every change)** drives **long deploy times** and makes deploys **highly exposed to transient network failures** (every build re-downloads deps like Electron and re-fetches all entries).
- **Recommendation:** move to **SSR with dynamic cache revalidation + cache priming** — cuts deployment overhead, improves content-update efficiency, and scales better. **Immediate optimization:** switch the **Launch configuration to Astro**; Contentstack concurrently addressing **Azure network hiccups**.

## Generalized guidance (for future similar queries)

- **"Frequent / intermittent deploy failures, especially on a large SSG site"** → the **full-rebuild-per-change SSG model amplifies transient network failures** (dependency downloads, fetching all entries). The durable fix is **SSR + cache revalidation + cache priming**, not just retrying.
- **Fail right after "Cloning Repository"** → suspect **GitHub connection / OAuth token**; reconnect the GitHub app + **Repair Connection**.
- **`electron` (or any large-binary dep) `socket hang up` during install** → avoid downloading big binaries each build (remove if unneeded / use prebuilt) to reduce transient install failures.
- **"Logs start mid-deployment"** → just the **5,000-line display cap**, earlier steps ran fine.
- **"Successful deploy but missing content / site LIVE with gaps"** → check whether the **build fetched all entries from CDA**; look for customer-side **`422`s (bad/deleted entry_uid, wrong-branch token)** and confirm publish to the right env/branch — separate from infra deploy failures.
- **Change project owner** = **backend transfer** (no UI); supply new-owner email + Org UID + Project UID.
- Related (same customer, separate threads): [CF001 site-slow CPU/mem](../logs-monitoring/launch-uva-cf001-site-slow-cpu-memory-100-redeploys-masked-add-caching.md), [Cloudflare 504 downtime](../logs-monitoring/launch-downtime-cloudflare-504-retry-redeploy-proactive-guard.md), [intermittent deploy fail — invalid env var](launch-intermittent-deploy-failure-invalid-env-var-config-poor-logs.md).

## Status

- **Multi-issue, ongoing.** Resolved/explained: **logs = 5k-line cap**; **clone-step failures = GitHub OAuth/connection** (reconnect + Repair); **owner change = backend transfer** (done for `baa5d@uvahealth.org`); **electron `socket hang up` = large-binary download + transient Azure network** (remove/prebuilt; Cloud investigating, **CLOUD-16476**). **Missing-content-at-build** still under joint Launch+CDA review (customer-side `422`s noted; no CDA 5xx). **Architectural recommendation issued:** **SSG full-rebuilds → SSR + cache revalidation/priming, switch config to Astro**; ticket left open pending customer's internal decision (CL-2863).
