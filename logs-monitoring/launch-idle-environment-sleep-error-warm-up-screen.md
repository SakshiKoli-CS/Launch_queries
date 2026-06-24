QUESTION

Users reported an **error when accessing a Launch environment that hasn't been used for a while** — the first load shows an error/blank, and after **reloading a few times / waiting a few seconds** it works. Observed that **projects "go to sleep"** if untouched for **~1–2 weeks**, affecting **every** such project; it appears both in **Visual Builder** and on the environment's **own URL**. Bad first impression, especially in front of a customer.

_Keywords: environment sleep, project goes to sleep, idle environment error, cold start error page, reload and it works, first access after inactivity, Environment Warm-Up Screen, LAUNCH-104, waking up screen._

ANSWER

## Cause — idle environments sleep (cold start on first access)

- Launch environments **go to sleep when idle** for an extended period (~1–2 weeks). The **first request after a long idle** triggers a **cold start**; until the environment wakes, the initial load can **error**, then succeed on reload once it's warm. This is expected idle behavior, not a per-project bug.

## Resolution — Environment Warm-Up Screen (now LIVE)

- The fix for the **poor first-load experience** is the **Environment Warm-Up Screen** (roadmap item **LAUNCH-104**), which is **now live**.
- Instead of an error/blank on first access to a sleeping environment, users now see a **friendly "warming up" screen** while the environment wakes, then the site loads — no more confusing error + manual reloads in front of customers.

## Keeping environments warm (avoid the wait on production)

- For production / low-traffic sites where you don't want cold starts at all, keep a container warm with an **external uptime check (e.g. Pingdom)** — see [launch-cold-starts-idle-container-keep-warm-pingdom.md](launch-cold-starts-idle-container-keep-warm-pingdom.md). (Note: that entry covers the **~5-minute** idle-termination cold start; the multi-week "sleep" here is the longer-idle case the Warm-Up Screen now handles gracefully.)

## Generalized guidance (for future similar queries)

- **"Error/blank when opening a Launch environment that's been idle a while; reload fixes it"** → the environment **went to sleep**; first access cold-starts it. There's now an **Environment Warm-Up Screen (LAUNCH-104, live)** that shows a waking-up screen instead of an error.
- **To avoid the wait entirely** on important environments, **keep them warm** with a periodic uptime check (Pingdom).
- Affects **any** long-idle environment, in Visual Builder and on the direct URL.

## Status

- **Resolved by the Environment Warm-Up Screen (LAUNCH-104), now live** — a sleeping environment now shows a warm-up screen on first access instead of an error.
