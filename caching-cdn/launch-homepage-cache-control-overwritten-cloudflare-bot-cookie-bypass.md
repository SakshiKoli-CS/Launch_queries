QUESTION

Customer **First Interstate Bank** (Org `blt6eec7c5cd96cdb6b`, AWS NA, SF case **00057600**) implemented ISR/SWR caching per Launch docs. The expected header `max-age=0, s-maxage=60, stale-while-revalidate` applied to **all pages except the home page (`/`)**. On `/` only, `Cache-Control` was **overwritten** with `no-cache, no-store, must-revalidate`.

Evidence (curl):
```
/         → cf-cache-status: BYPASS   cache-control: no-cache, no-store, must-revalidate   x-cache-control: max-age=0, s-maxage=60, stale-while-revalidate
/support  → cf-cache-status: MISS→HIT cache-control: max-age=0, s-maxage=60, stale-while-revalidate
```
The **`x-cache-control`** debug header (set by the app to the same value) confirmed the **app emits the correct header** — it's being **overridden downstream (CDN/infra)**, on `/` only. They asked whether Launch/Cloudflare applies special rules to the root path.

_Keywords: cache-control overwritten, home page no-store, root path / not caching, cf-cache-status BYPASS, x-cache-control debug, SWR not working homepage, __cf_bm cookie, Cloudflare bot management cache bypass, ISR SWR Launch, API usage spike homepage._

ANSWER

## Root cause — Cloudflare anti-bot cookie (`__cf_bm`) forcing a cache BYPASS on `/`

- The override was **not** a Launch/Cloudflare rule specific to the homepage in the usual sense, and **not** the application. It traced to **Cloudflare bot management**: an **anti-bot cookie (`__cf_bm`) appeared on the home page only**, which **triggered a cache BYPASS** (Cloudflare treating it as suspected-bot traffic and serving uncacheable). The homepage typically has **stricter bot-detection** behavior, which is why only `/` was affected.
- Notably, even after **Cloudflare bot management was turned off, the cookie was still being sent**, so it required **Cloudflare-side investigation** (escalated; tracked as **CL-3748** + a Cloudflare support ticket).

## The initial Next.js hypothesis was WRONG (and how it was ruled out)

- First hypothesis: Next.js App Router auto-marking `/` **dynamic** (via `cookies()`/`headers()`/`force-dynamic`/`revalidate:0`/middleware) → forcing `no-store`.
- The customer **disproved** this: running the same app **locally in production mode** (`pnpm build && pnpm start`) returned the **correct** `Cache-Control` on `/`. So the app emits the right headers; the override happens **only when running behind Launch's edge/CDN**. They also noted they use `force-dynamic` + explicit headers **everywhere** per Launch docs, so `/` isn't special at the app layer.
- **Lesson:** the `x-cache-control` (correct) vs `cache-control` (overwritten) split + a **clean local prod build** proves the rewrite is **downstream**, not the app — don't stop at "Next.js marked it dynamic."

## Resolution

- After the Cloudflare-side fix, the **override stopped**: `/` now honors `max-age=0, s-maxage=60, stale-while-revalidate` with normal Cloudflare progression (MISS → HIT). **Fixed in production.**
- Business impact resolved: previously `/` hit the **Content API on every request** (no cache) pushing the customer **over API limits**; after the fix, **API usage dropped sharply**.

## Generalized guidance (for future similar queries)

- **"Cache-Control overwritten to no-store on one page (often `/`) but correct elsewhere; cf-cache-status: BYPASS"** → suspect **Cloudflare bot management / `__cf_bm` cookie** forcing a BYPASS for suspected-bot traffic (homepages get stricter bot rules). Escalate to Cloudflare; the app likely isn't the cause.
- **Prove app vs downstream:** set a **duplicate debug header** (e.g. `x-cache-control`) to the intended value, and run a **local prod build** (`build` + `start`). If the app/local headers are correct but the edge response differs, the **rewrite is downstream (CDN/infra)**.
- **Don't over-attribute to Next.js "dynamic route"** — `force-dynamic` + explicit `Cache-Control` (per Launch ISR/SWR docs) is the documented pattern; a BYPASS on one route is more likely **edge/bot-management**, not app rendering.
- **Watch the cost signal:** a route stuck uncacheable hammers the **Content Delivery API** and can blow API limits — a sudden API-usage spike on one path corroborates a caching BYPASS.

## Status

- **Resolved in production** — Cloudflare-side fix (bot-cookie/BYPASS) applied; `/` now caches per ISR/SWR config (MISS→HIT); customer validated headers and saw a large dip in API usage. Tracked CL-3748.
