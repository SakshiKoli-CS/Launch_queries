QUESTION

Customer **Sophos** (1 Launch Project + 2 Small Launch Projects) asks about **Launch Cloud Functions** pricing/limits:
- Any **execution limits, per-request charges, or usage-based pricing** beyond their existing subscription?
- Any **rate limits** to flag for **high form-submission volumes**?
- Any other **pricing considerations**?
- And a follow-up: is the **GB-hours compute allowance monthly, yearly, or lifetime**?

_Keywords: Cloud Functions pricing limits, per-request charges, usage-based pricing, rate limits form submissions, GB-hours serverless compute time, 500 GB-hours monthly, split 250 build 250 serverless, fair usage no hard limit, aggregated across projects, entitlement 1 project 2 small projects, Cloud Functions vs API routes SSR recommendation._

ANSWER

## Compute allowance — GB-hours, MONTHLY, and it's SPLIT between build and serverless

- Entitlement for this account (**1 Launch Project + 2 Small Launch Projects**) = **500 GB-hours of compute time per MONTH**, aggregated across all projects.
- ⚠️ **Important correction:** the 500 GB-hours is **NOT purely serverless-function time.** It **includes both build time and serverless-function time** — **250 GB-hours build + 250 GB-hours serverless per month**. Don't quote "500 GB-hours of serverless functions."
- **Period = MONTHLY** (not yearly, not lifetime).

## Limits / charges

- **No per-request charges, no usage-based pricing** beyond the existing subscription.
- **No additional rate limits** to flag (including for **high form-submission volumes**) — treated under **fair usage**.
- **Fair usage, soft limit:** exceeding the allowance is **not hard-enforced** — Launch doesn't cut them off if they go over the GB-hours under fair use.

## Guidance — Cloud Functions vs. API routes (SSR)

- **Cloud Functions are mainly for STATIC sites on Launch** that still need some server-side compute.
- For customers on a **server framework (e.g. Next.js with SSR)**, **create API routes directly in the application** instead — Launch **scales those routes just as effectively**, and **using Cloud Functions in that case is discouraged.**

## Generalized guidance (for future similar queries)

- **Cloud Functions pricing** → **no per-request/usage-based charges beyond subscription; no extra rate limits** (incl. high form volumes); governed by a **monthly GB-hours compute allowance under fair usage** (soft, not hard-enforced).
- **The GB-hours allowance is MONTHLY and SPLIT** — e.g. 500 GB-hours = **250 build + 250 serverless** per month; it is **aggregated across all the account's projects**. **Don't represent the whole figure as serverless-only.**
- **Architecture steer:** **Cloud Functions = for static sites needing some server compute.** On **SSR frameworks (Next.js), use in-app API routes** — Launch scales them; Cloud Functions are discouraged there.
- Related: [Cloud Functions monorepo setup](launch-cloud-functions-monorepo.md), [Cloud Functions node_modules / dependencies](launch-cloud-functions-node-modules-dependencies.md), [Cloud Functions long path](launch-cf-long-path.md), [load/performance testing must be pre-approved (same load-test context)](../logs-monitoring/launch-load-performance-testing-allowed-must-be-preapproved-checklist.md), [no static IP / secure DB-API via Cloud Functions](../networking-connectivity/launch-no-static-ip-no-vpn-secure-db-api-via-auth-cloud-functions-azure.md).

## Status

- **Answered (with correction).** **No per-request/usage-based charges or extra rate limits** beyond subscription — **fair usage, no hard enforcement**. Compute allowance is **MONTHLY GB-hours**, **aggregated across projects**, and **split between build and serverless** (this account: **500 GB-hours/mo = 250 build + 250 serverless**) — **not 500 GB-hours of serverless alone** (initial answer corrected). **Cloud Functions are for static sites needing server compute; SSR (Next.js) customers should use in-app API routes** instead.
