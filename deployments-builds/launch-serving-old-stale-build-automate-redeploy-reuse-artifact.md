QUESTION

Customer **MicroStrategy** (SF case **00057329**, "Launch is serving old builds & weird behavior", Project `6765d82c9f1088499e8a2820`, Stack `bltf8d808d9b8cebd37`, Org `blt45a2a52367b47001`) reported the website **intermittently serving OLDER code**:

- Verified on the **direct Launch origin URL** (so **not** customer-side caching).
- Example: Build **1307** deployed fine; next morning Build **1314** was active but **showing old code**. **Downloading the code for both builds shows the correct code** — yet the served site shows the wrong/old version. A "**successful build**" could serve **missing code**.
- They run an **Automate job that auto-redeploys Production whenever a Contentstack release is deployed** (hence extra overnight builds). These Automate builds **reuse the last uploaded artifact** (fresh upload only when there's new code). The automation has worked for **>1 year**; they **suspect the Automate→Launch trigger broke**.
- Also seeing **higher-than-normal build failures** during dependency install (rebuild fixes it) — *see separate root cause below*.

_Keywords: serving old build, stale code served, old code on origin, correct build wrong website, successful build missing code, Automate auto-redeploy, reuse last uploaded artifact, redeploy trigger, intermittent, not caching._

ANSWER

## The two issues in this case (keep them separate)

1. **Website serving old/stale code despite a correct active build** — the primary "weird behavior" (this entry).
2. **Intermittent build failures during dependency install** — traced to the **`sharp` postinstall `ECONNRESET`** (sharp < 0.33 downloading libvips). Fix = upgrade to `sharp@^0.33.5`. Full detail: [launch-deployment-econnreset-sharp-libvips-upgrade.md](launch-deployment-econnreset-sharp-libvips-upgrade.md). (The customer's "rebuild fixes it, tied to dependency installation, not code/ESLint" matches this.)

## Stale-build issue — what was established

- The served code was **wrong even on the Launch origin URL**, ruling out **customer-side CDN caching**.
- The **downloaded artifacts for the builds were correct**, so the discrepancy is between the **artifact and what's actually being served** for those Automate-triggered redeploys.
- Strong correlation with the **Automate auto-redeploy** flow: these builds **reuse the last uploaded file** rather than a fresh upload, and the problem appeared on **overnight Automate-triggered builds**. **Disabling the Automate job stopped recurrence; re-enabling brought it back** — pointing at the **release-triggered auto-redeploy (reusing the last artifact)** as the trigger.

## Investigation status / labeled uncertainty

- **No definitive Launch-side root cause was confirmed in-thread.** Suspicion centered on the **Automate→Launch redeploy reusing the last artifact** combined with the **intermittent build failures** (sharp), so a redeploy could land in a bad/partial state and serve stale output.
- Investigation was **blocked on code/git access** — the customer could share only a **zip snapshot** (not git history), and a point-in-time snapshot couldn't show **changes over time** (a config/code change around **~Apr 7–8** was a hypothesis, unconfirmed).
- **Mitigation that worked:** **disable the Automate auto-redeploy** job to stop the stale-code recurrence while investigating. Fixing the **build failures (sharp upgrade)** removes the intermittent-failure amplifier.

## Generalized guidance (for future similar queries)

- **"Site serves old/stale code but the active build (and downloaded artifact) is correct"** → first confirm it reproduces on the **origin URL** (rules out customer caching). If yes, look at **how the deployment was triggered** — especially **Automate auto-redeploys that reuse the last uploaded artifact** rather than a fresh upload.
- **Correlate with the trigger:** if **disabling an Automate redeploy job stops it** and re-enabling reproduces it, the **release-triggered redeploy** is implicated.
- **Don't conflate** a co-occurring **build-failure** issue with the **stale-serve** issue — here the failures were **sharp/ECONNRESET** (separate, fixable); but intermittent failed/partial builds can make a reused-artifact redeploy serve stale output, so fix both.
- **For time-based "what changed" analysis**, a single **zip snapshot isn't enough** — request **read-only git access** to see changes over time (set expectations that point-in-time code can't reveal regressions).

## Status

- Build-failure portion: **resolved** via `sharp@^0.33.5` upgrade (separate entry). Stale-build behavior: **mitigated** by disabling the Automate auto-redeploy; deeper root-cause analysis **blocked on code/git access** (customer declined). Note: across MicroStrategy's recent P1s, root causes have repeatedly been **app/config-side** — factor that into triage.
