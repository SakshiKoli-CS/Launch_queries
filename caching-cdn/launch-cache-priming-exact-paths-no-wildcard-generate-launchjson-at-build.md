QUESTION

Customer (Sheril Subramanian) has questions about **Prime Caching** in `launch.json`:

1. Cache-priming paths must be **relative** — if they add `/blog`, does that cache **all blog articles** (`/blog/article1`, `/blog/article2`, …)?
2. Since **regex and wildcards are not supported**, is there an **easier way to add many URLs** for a large site — e.g. **generate the list at build/run time** and write it into `launch.json`?

_Keywords: prime caching, cachePriming urls launch.json, relative path, no wildcard no regex, /blog does not cache articles, generate launch.json at build time, dynamic routes generateStaticParams, large site many URLs, cache priming flag per page, group-level configuration._

ANSWER

## 1. `/blog` only primes `/blog` itself — NOT the article pages

- Adding **`/blog`** to `launch.json` **cache primes only the `/blog` page itself.** It does **not** automatically prime dynamic article pages like **`/blog/article1`** or **`/blog/article2`.**
- Cache priming uses **exact URLs** — **no wildcard / no regex / no prefix matching.** Each URL you want primed must be listed explicitly.

## 2. Easier way for many URLs — generate `launch.json` at build time

Since you can't wildcard, **generate the `cachePriming.urls` list automatically during the build** so you don't hand-maintain a large list.

**Step 1 — mark pages to prime.** In each page you want primed, export a flag:
```js
export const cachePrimingEnabled = true;
```

**Step 2 — add a generator script** (e.g. `generate-launchjson.js`) that scans the app dir for pages with that flag and writes `launch.json`. For **dynamic routes** (`[slug]`), it resolves the **concrete pre-rendered URLs from the build output** (`.next/server/app` for App Router, `.next/server/pages` for Pages Router):
```js
const fs = require("fs");
const path = require("path");

const CACHE_FLAG = /export\s+const\s+cachePrimingEnabled\s*=\s*true/;
const DYNAMIC_SEGMENT = /\[.+?\]/;

function findFiles(dir, match) {
  if (!fs.existsSync(dir)) return [];
  return fs.readdirSync(dir, { withFileTypes: true }).flatMap((entry) => {
    const full = path.join(dir, entry.name);
    return entry.isDirectory() ? findFiles(full, match) : match(entry.name) ? [full] : [];
  });
}

function pagePathToUrl(filePath) {
  const dir = path.dirname(path.relative(path.join(__dirname, "app"), filePath));
  return dir === "." ? "/" : `/${dir.replace(/\\/g, "/")}`;
}

function resolveUrlsFromBuildOutput(routePattern) {
  const routeRegex = new RegExp(
    "^" + routePattern.replace(/\[(?:\.\.\.)?\w+\]/g, "[^/]+") + "$"
  );
  const seen = new Set();
  return [
    path.join(__dirname, ".next", "server", "app"),   // App Router
    path.join(__dirname, ".next", "server", "pages"),  // Pages Router
  ].flatMap((buildDir) =>
    findFiles(buildDir, (name) => name.endsWith(".html")).reduce((acc, file) => {
      let url = "/" + path.relative(buildDir, file).replace(/\\/g, "/").replace(/\.html$/, "");
      if (url.endsWith("/index")) url = url.slice(0, -6) || "/";
      if (routeRegex.test(url) && !seen.has(url)) { seen.add(url); acc.push(url); }
      return acc;
    }, [])
  );
}

const generatedUrls = findFiles(path.join(__dirname, "app"), (name) => /^page\.(tsx|ts|jsx|js)$/.test(name))
  .filter((file) => CACHE_FLAG.test(fs.readFileSync(file, "utf8")))
  .flatMap((file) => {
    const url = pagePathToUrl(file);
    if (!DYNAMIC_SEGMENT.test(url)) return [url];
    const resolved = resolveUrlsFromBuildOutput(url);
    if (resolved.length === 0) console.warn(`No pre-rendered pages found for ${url} — ensure generateStaticParams is exported.`);
    return resolved;
  });

const launchJsonPath = path.join(__dirname, "launch.json");
const existing = fs.existsSync(launchJsonPath)
  ? JSON.parse(fs.readFileSync(launchJsonPath, "utf8"))
  : {};
const existingUrls = existing?.cache?.cachePriming?.urls ?? [];
const urls = [...new Set([...existingUrls, ...generatedUrls])];

fs.writeFileSync(launchJsonPath, JSON.stringify({ cache: { cachePriming: { urls } } }, null, 2) + "\n");
console.log(`Generated launch.json with ${urls.length} URL(s):`, urls);
```

**Step 3 — wire into the build** (`package.json`):
```json
"build": "next build && node generate-launchjson"
```

**Step 4 — redeploy.** `launch.json` is regenerated each build with the full URL list — no manual maintenance.

Notes:
- Dynamic routes only resolve if the pages are **pre-rendered** — make sure **`generateStaticParams`** is exported (the script warns when it finds none).
- The script **merges with existing `cachePriming.urls`** and de-dupes.

## Group-level / metadata-driven priming (customer idea)

- The customer wanted to attach the priming flag at a **group level (e.g. via Taxonomy metadata in the CMS)** rather than per page. That's a reasonable enhancement; the suggested starting point is the **cache-priming GitHub example repository** (adapt its code for group/metadata-driven selection).

## Generalized guidance (for future similar queries)

- **"Does a parent path prime its children?"** → **No.** Cache priming is **exact-URL only — no wildcards, no regex, no prefix expansion.** `/blog` primes `/blog`, not `/blog/*`.
- **"How do I prime many URLs without hand-maintaining the list?"** → **generate `cachePriming.urls` at build time**: tag pages with a flag, scan for the flag, expand **dynamic routes from the pre-rendered build output**, write `launch.json`, run it after `next build`. Requires `generateStaticParams` for dynamic routes.
- Related: [Revalidate CDN Cache — one param per request](launch-revalidate-cdn-cache-one-parameter-per-request.md), [cache priming failures / rate limits](launch-cache-priming-failures-rate-limits-batching.md).

## Status

- **Answered.** Confirmed **`/blog` primes only `/blog`** (no auto-priming of articles; no wildcards/regex), and provided a **build-time `launch.json` generator** plus the **GitHub example repo** for group/metadata-driven selection. Awaiting customer confirmation to close.
