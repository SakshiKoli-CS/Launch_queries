QUESTION

Does **site traffic / runtime usage** in Launch get **consumed/deducted from the monthly data transfer** amount? (i.e. how do build time, server execution, and data transfer limits relate?)

_Keywords: site traffic runtime usage, data transfer quota, build time limit, server execution limit, hard limit vs soft limit, monthly reset, contract-specific limits, product analytics over-consumed, plan upgrade, runtime vs build hours._

ANSWER

## The three limits and how they behave

All of these are **contract-specific** and vary by account:

- **Build time limit — HARD limit.** Once reached, **new deployments are blocked** until it **resets (monthly)** or the **plan is upgraded.**
- **Server execution limit — SOFT limit.** Even if exceeded, **the site keeps running**, but usage shows as **over-consumed in Product Analytics.** To avoid that, **increase the limit via a plan upgrade.**
- **Data transfer vs runtime.** **Site traffic / runtime usage is tracked separately from build time** and **contributes to server execution usage, not build hours.** Whether it **maps to a data transfer quota depends on the customer's contract.**

## Direct answer

- **Runtime/site-traffic usage counts toward server execution**, *not* build time. Whether it **also** draws down a **data transfer** quota is **contract-dependent** — check the specific account's contract.

## Generalized guidance (for future similar queries)

- **"Does traffic/runtime consume my data transfer / build time?"** → **Runtime ≠ build time.** Runtime/traffic → **server execution usage**; **build time** is its own bucket. **Data-transfer mapping is contract-specific.**
- **Hard vs soft:** **build time is a hard cap** (blocks deploys when hit; resets monthly or on upgrade); **server execution is soft** (site stays up, just shows over-consumed in Product Analytics → upgrade to clear it).
- All three limits are **per-contract** — don't quote fixed numbers; confirm against the account's plan.

## Status

- **Answered.** Build time = **hard** (blocks deploys, monthly reset/upgrade); server execution = **soft** (site runs, shows over-consumed); **runtime/traffic counts toward server execution**, and **data-transfer mapping is contract-specific.** Customer satisfied.
