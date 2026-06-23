QUESTION

Customer **Kolesa Group** reported deployment issues with their Contentstack Launch project after adding **Contentstack Live Preview** (`@contentstack/live-preview-utils`) to their codebase. They have 3 CTS Launch environments; the **staging** deployment was stuck in **"Queued"**, and a prior attempt had failed with:

> "Deployment failed: Please try to redeploy the site, or contact support for assistance."

Two distinct problems surfaced:
1. **Deployments** — failed/cancelled around the reported timeframe, then stuck queued.
2. **Live Preview** — even after a successful deploy of the Live Preview codebase, **Live Preview did not work**, intermittently showing **"Live Preview SDK Not Initialized"** or appearing not enabled (which the customer said was not true).

Is this a Launch-side issue or a build/application failure tied to the recent Live Preview changes?

- **Launch Project ID:** `66a287b352edf03fcf2b91e4`
- **Deployment UIDs (all include Live Preview changes unless noted):**
  - `6a3289b613d7373b494ff48e` — LIVE
  - `6a326ca2ab61d2ee2438ea19` — Archived
  - `6a326aef68a44015549b4ec7` — Failed
  - `6a3268ed68a44015549b4d86` — Cancelled
  - `6a3266b813d7373b494fe49e` — Cancelled
  - `6a3256c381c4bcfd58c8b31b` — Archived (before Live Preview changes)

ANSWER

**Investigation**
- Deployment history showed deployments were **cancelled twice** around the reported timeframe. Requested the failed **Deployment UID** to correlate with backend logs.
- The staging environment with the latest Live Preview codebase **did eventually deploy successfully** (UID `6a3289b613d7373b494ff48e` went LIVE), so the persisting problem shifted from deploy failure to **Live Preview not functioning**.

**Live Preview / app-side observations**
- From the screenshots, the **website was not loading inside the iframe**, which points to a **CORS / CSP issue** rather than a Launch deployment fault.
- Symptoms reported: intermittent **"Live Preview SDK Not Initialized"** and Live Preview showing as not enabled.
- **Suggested check:** open the **browser console** when accessing Live Preview and look for **CSP errors**. If present, refer to the Visual Experience docs — **"Could Not Connect to Website"** section.
- Visual Experience team was looped in to validate the Live Preview changes.

**Access note (delay during investigation)**
- Customer added the support engineers as project admins, but login kept **redirecting to the SSO flow**, blocking access. Resolved after the customer **disabled SSO-only login and re-sent invitations** — access then worked.

**Resolution**
- Issue **resolved and case closed** (confirmed by the customer-facing team). Deployment reached LIVE; remaining Live Preview behavior was handled as a client-side CORS/CSP configuration matter.

**Suggested checks (for similar reports)**
- Separate the two failure modes: a stuck/failed **deployment** vs. **Live Preview not rendering** — they have different root causes.
- For Live Preview iframe failures, check **CSP/CORS headers** and browser console before assuming a Launch-side deploy issue.
- Ensure support access isn't blocked by **SSO-only enforcement** when sharing org/project access.
