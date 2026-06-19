QUESTION

Customer **Remote.com** raised two questions about their Launch project (EU):

1. **How can they see the full list of pages generated at build time** (the pages that load instantly when visited), ideally per deployment?
2. **More serious concern:** they generate a **`launch.json`** at build time to set up **critical rewrites and other deployment configs**. They observed that **when a deployment fails, these configs appear to be lost / no longer applied**. Previously with Vercel, only a **successful** deployment changed the config — a failed one left the previous config in place. How can they get the same behavior on Launch?

ANSWER

**Q1 — pages generated at build time**
- Important distinction: **generated at build time ≠ cached.** Pages being pre-rendered at build does not by itself guarantee they're warm in cache.
- To guarantee pages load instantly, the customer should use **cache priming**.
- (Customer self-resolved this question before we responded.)

**Q2 — `launch.json` rewrites lost on a failed deployment**
- **Expected behavior:** if a deployment **fails**, changes to `launch.json` are **not applied** — the **last successful deployment's config remains in effect**. This is actually the same model the customer wanted from Vercel (failed deploy keeps prior config).
- The likely confusion: they **generate `launch.json` during the build**, so on a failed build the file may not be produced/preserved — but the previously deployed config is **not wiped**, it stays active.
- **Requirement:** ensure `launch.json` is created at the **root level of the project**.
- Reference: **Edge URL Rewrites** — https://www.contentstack.com/docs/developers/launch/edge-url-rewrites

**Suggested checks (for similar reports)**
- Reassure customers that a **failed deployment does not overwrite live config**; the prior successful deployment's rewrites stay in effect.
- Confirm `launch.json` is at the **project root** and is being generated reliably by the build step.
- For "instant load" / pre-render guarantees, point to **cache priming**, not just build-time generation.

**Status**
- Guidance provided; awaiting customer confirmation (follow-ups sent, no response yet at last update).
