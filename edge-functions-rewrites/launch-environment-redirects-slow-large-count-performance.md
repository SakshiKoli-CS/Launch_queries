QUESTION

Customer reports **environment-level redirects are extremely slow**. Under a specific environment path (e.g. `/page`) they have **~300 redirects** configured (high), and suspect the **count** is causing the performance issue. Does this number of redirects impact performance? Are there **known limitations or best practices** for redirects at scale?

_Keywords: redirects slow, too many redirects, 300 redirects performance, environment-level redirects, launch.json redirects, Edge Function redirects, redirect performance best practices, redirect lookup efficiency._

ANSWER

## First, confirm WHERE the redirects are configured

The performance characteristics depend on the implementation, so the key diagnostic question is whether the redirects are defined in:
- **`launch.json` (Edge URL Redirects)** — declarative path redirects evaluated at the edge, or
- **Edge Functions** — custom code performing the redirect logic.

Getting the **list of redirects** and the **mechanism** is needed to pinpoint the slowness. _(This case was awaiting those details — root cause not yet confirmed.)_

## Why a large redirect set can be slow (general guidance)

- **How they're evaluated matters more than the raw count.** In an **Edge Function**, if redirects are matched by **iterating/linear-scanning a large list per request**, ~300 entries can add measurable per-request latency. A **map/object keyed by path** (O(1) lookup) is far faster than a loop.
- **`launch.json` Edge URL Redirects** are handled declaratively at the edge and are generally more efficient for **static path→path** redirects than hand-rolled per-request logic.

## Best practices for redirects at scale

- Prefer **`launch.json` / Edge URL Redirects** for **static path redirects** rather than implementing them in Edge Function code.
- If using an **Edge Function**, use an **efficient lookup** (object/Map keyed by the request path) — **avoid iterating the whole list** on every request; avoid heavy per-request loops/regex over hundreds of entries.
- Keep the redirect set lean — remove stale entries; consolidate patterns where possible.

## Generalized guidance (for future similar queries)

- **"Redirects are slow / does a large number hurt performance?"** → First determine the **mechanism** (`launch.json` vs Edge Function). The **count alone** isn't the issue as much as **how matches are evaluated** — a **linear scan in an Edge Function** over hundreds of redirects is the usual culprit; switch to **O(1) path lookup** or move to **`launch.json` Edge URL Redirects**.
- Request the **redirect list + config location** to investigate; reproduce timing per redirect path.

## Status

- **Open / investigation pending** — requested confirmation of **where the redirects are configured** (`launch.json` vs Edge Functions) and the **redirect list**; awaiting customer. Root cause not yet confirmed; guidance above is general best practice for large redirect sets.
