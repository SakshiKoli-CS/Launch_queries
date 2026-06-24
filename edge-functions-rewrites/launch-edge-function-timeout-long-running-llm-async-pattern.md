QUESTION

Customer has an **Edge Function that needs to run 3–4 minutes** (it calls an LLM — Claude — to build a competitive-intel report; responses can take up to ~4 min) and it's **timing out** earlier:
```
[EDGE] [strategist] HTTP 524 from Claude: error code: 524
[EDGE] [pipeline] Claude strategy failed: 524
```
Can the function's **running time be increased**? They'd ideally want **5 min**.

_Keywords: edge function timeout, increase edge function runtime, 120s default timeout, 5 minute max, 524 LLM call, long-running edge function, call and wait, async processing pattern, poll for result, offload to origin, streaming response._

ANSWER

## Edge Function timeout limits

- **Default Edge Function timeout: 120 s (2 min).** It can be **increased up to 5 minutes**, but this has **cost implications** and requires **approval + analysis** (tech and cost) — it's **not enabled instantly**.

## But: a 3–4 min blocking edge function isn't best practice

- **Long-running functions at the edge are discouraged** — they impact **scalability and performance**. Recommended to **optimize to reduce response time** or **offload long processing to the origin**, rather than holding an edge function open for minutes.

## The real decision — which API pattern is it?

Two behaviors, handled very differently:
1. **Responds quickly (first byte early), then streams** data continuously for minutes → evaluate **long-lived streaming** support (connection stays active, data keeps flowing — not idle).
2. **No response for 3–4 min, returns only when processing completes** ("call and wait") → platforms (incl. Launch) **enforce limits on how long they wait for an idle upstream with no data**. The standard pattern is **asynchronous**:
   - **accept the request**,
   - **trigger processing asynchronously**,
   - **fetch the result later** (polling, callbacks, or stream once ready).

**This customer's case = #2** (call-and-wait, JSON response, no streaming needed). So the right approach is the **async pattern**, not merely bumping the timeout — a single synchronous call held open for 4 minutes is fragile (hence the 524). Bumping to 5 min is a stopgap at best.

## Generalized guidance (for future similar queries)

- **"Can I increase the Edge Function timeout?"** → default **120 s**, raisable to **5 min** with **cost approval/analysis** (not instant). But for **multi-minute work, don't** — it's not the edge's design point.
- **524 on a long upstream (LLM) call from an edge function** → the upstream took too long with no data flowing; the platform timed out the idle wait.
- **For long "call-and-wait" (non-streaming) work** → use an **async pattern**: accept → process asynchronously → return result via **polling/callback** later. Or **offload to the origin** (Cloud Functions / a backend) rather than the edge.
- **For long streaming responses** → ensure the response **streams early and continuously** (no long idle gaps) and that the platform supports long-lived streaming.

## Status

- Clarified: edge timeout is **120 s (→ up to 5 min with cost approval/analysis)**, and a 3–4 min synchronous edge call **isn't the recommended pattern**. Customer's case is **call-and-wait (#2)** → guided toward an **async processing** approach (and possibly origin offload) rather than relying on a longer edge timeout.
