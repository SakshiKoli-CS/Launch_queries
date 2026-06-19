QUESTION

Customer **Remote.com** (Case **00059498**, urgent — launching in a few hours) reported that some requests **return blank pages**. Server logs from **Deployment #83** showed errors, and the customer attached `server-logs.txt`. Next.js version **16.2.6**, app path `apps/website`, EU stack.

The blank pages correlated with a **Next.js cache write failure** in the server logs, plus a separate **"locale / language not found"** error the customer also wanted explained. Is this Launch-side, and what can they change before launch?

ANSWER

**Symptoms**
- Some requests return **blank** responses.
- Server logs show a `.next/cache` **write failure** (`ENOENT`) at `/var/task/sourcecode/apps/website/.next/cache`.
- Separate recurring **"language/locale not found"** error.

**Cause (primary — blank pages)**
- Next.js was trying to create its cache directory under **`/var/task/...`**, which is **read-only** in the Launch (Lambda-style) runtime **post-deploy**.
- This is a **read-only filesystem** problem, **not disk space** — a full disk reports `ENOSPC`, whereas the logs showed `ENOENT`.
- With the cache unwritable, **image optimization and page caching both fail**, producing blank pages.

**Fix (cache write failure)**
- Configure Next.js to write its cache to a **writable path** — `/tmp` or `/tmp/cache`.
- The customer's first attempt at relocating the cache **wasn't respected** (cache errors persisted on a follow-up deployment).
- It worked after they added a custom handler via **`cacheHandlers`** (plural) — see Next.js docs: `next-config-js/cacheHandlers`. After this, **no errors displayed and navigation felt snappy**.
- **Verified:** by **Deployment #103** the **`ENOENT` cache error was resolved**.

**Locale "not found" error**
- Customer confirmed all enabled repository locales are enabled in Contentstack and asked **which locale** was reported missing.
- This is **not Launch-specific** — routed to the **CDA team** to identify the specific language from internal logs. (Remained open in this thread.)

**Caching / performance guidance**
- All **blog pages are pre-rendered at build time**, so the blog path should show a **high cache-hit ratio** (a cache-hit/miss summary for deployment #103 was shared with the customer).
- **Optimization:** increase **`s-maxage`** beyond the current **60s** (e.g. `3600`+, depending on content update frequency) so CDN/edge caches serve longer without revalidation — meaningful for relatively static content like blog posts.
- Customer also asked about **enabling compression** for delivered assets (open question).

**Resolution status**
- Cache `ENOENT` / blank-page issue **resolved** by deployment #103 (cache relocated via `cacheHandlers`).
- **Open:** locale-not-found (with CDA), compression request, and confirmation of cache-hit visibility — awaiting customer response; team followed up.

**Suggested checks (for similar reports)**
- Blank pages + `.next/cache` errors on Launch → confirm `ENOENT` vs `ENOSPC`: `ENOENT` = read-only FS, relocate cache to `/tmp`.
- Prefer the **`cacheHandlers`** config when a simple cache-dir relocation isn't honored.
- For static/blog content, tune **`s-maxage`** to raise edge cache-hit ratio.
- Locale-not-found errors belong with the **CDA team**, not Launch — capture the specific failing locale.
