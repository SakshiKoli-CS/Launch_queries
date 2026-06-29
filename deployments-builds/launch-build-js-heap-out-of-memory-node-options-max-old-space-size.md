QUESTION

Customer **UVA** (Project `6800171649412f0653c0eb70`, Azure NA, SF case **00050632**) suddenly gets a **JavaScript heap out-of-memory error while building** one environment on Launch:

```
<--- Last few GCs --->
... Mark-Compact ... 4070.1 (4146.8) MB ... allocation failure; GC in old space requested
FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory
```

They resolved it by adding **`NODE_OPTIONS=--max-old-space-size=8192 npm run build`** — is that **safe/expected**, and what's the max?

_Keywords: JavaScript heap out of memory build, FATAL ERROR Reached heap limit allocation failed, Mark-Compact GC old space, NODE_OPTIONS max-old-space-size 8192 8GB, Node default memory 2-4GB, Next.js Webpack Vite build OOM, bundle size grown, safe workaround._

ANSWER

## Cause — Node ran out of heap during the build

- `FATAL ERROR: Reached heap limit … JavaScript heap out of memory` happens when **Node.js exhausts its heap during the build** (commonly **Next.js / Webpack / Vite** builds).
- By default Node caps heap at roughly **~2–4 GB** (depends on runtime/env).

## Fix — raise the heap with `NODE_OPTIONS=--max-old-space-size`

- **`NODE_OPTIONS=--max-old-space-size=8192`** raises the limit to **8 GB**, letting larger builds (many deps, large assets, heavy bundling) finish. This is a **safe workaround**, and **8 GB is the recommended safe ceiling** (the customer is already using it).
- **Don't set it arbitrarily high** — needing more than 8 GB usually signals a **deeper issue**, not a reason to keep bumping the number.

## Better — find & fix the root cause if it recurs

If builds **frequently** hit this, investigate rather than relying on the flag:
- **Bundle size grown substantially** (very large JS / asset imports).
- **Heavy build-tooling memory** (custom Webpack plugins, image optimizations, etc.).
- **Review build logs** for unusually large dependencies/bundles.

## Generalized guidance (for future similar queries)

- **`JavaScript heap out of memory` / `Reached heap limit` during a Launch build** → **set `NODE_OPTIONS=--max-old-space-size=8192`** (8 GB) on the build. Safe and expected; Node's default (~2–4 GB) is the constraint.
- **8 GB is the safe ceiling** — going higher hints at a real problem (bloated bundle, memory-heavy plugins). **Prefer fixing the root cause** (audit bundle size / build tooling) over ever-larger memory flags.
- (Not yet documented officially, but a standard Node build workaround.)
- Related: [build fails no error → L2 build machine](launch-build-failed-no-error-l2-build-machine.md), [usage limits / build compute](launch-usage-limits-build-time-server-execution-data-transfer.md), [scalability / large SSG builds](launch-scalability-compute-limits-large-ssg-builds-runtime-throughput-sla.md).

## Status

- **Resolved.** Build OOM (`Reached heap limit … heap out of memory`) fixed with **`NODE_OPTIONS=--max-old-space-size=8192`** — confirmed **safe**, **8 GB is the recommended ceiling**. Recommended the customer **find/fix the underlying memory growth** (bundle size / build tooling) rather than relying on the flag long-term.
