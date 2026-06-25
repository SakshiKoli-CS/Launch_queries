QUESTION

Customer **Arkansas Blue Cross Blue Shield** (Skai sites) wants to serve a **custom 404 page** instead of Contentstack/Launch's **default "not found" page**, for a **Nuxt 3 fully static (SSG) site**. Pages are generated to individual folders (e.g. `.output/public/skaibcbs`, `.output/public/skaibluecross`) and pointed to via Launch.

They're unsure whether to configure it in **`launch.json`**, or place a **`404.html`** in the static folder — docs don't cover the fully-static-Nuxt case.

_Keywords: custom 404 page, Nuxt 3 fully static SSG, prevent default not found page, 404.html static output, launch.json 404, edge function 404 fallback, correct HTTP 404 status SEO, nested routes 404, SSR vs static error page._

ANSWER

## Why the default 404 shows — Launch is a static CDN for SSG

- In **Nuxt 3 fully static (SSG)** mode, **Launch behaves as a static CDN**. If a requested route **doesn't physically exist** in the build output, Launch returns its **system 404 page before any Nuxt logic runs** — so the custom Nuxt 404 never gets a chance.

Two supported approaches:

## Option 1 — Edge-Level 404 handling (recommended for SSG)

Use a **Launch Edge Function** that lets Launch resolve the request normally, and **if Launch returns a 404**, **fetches the generated `/404.html`** and returns it **with an explicit HTTP 404 status**. This:

- Renders the **custom Nuxt 404 page**
- Preserves the **correct HTTP 404 status** (SEO-safe)
- Works for **nested routes** (e.g. `/foo/bar/baz`)
- Needs **no `launch.json` rewrites**, and keeps `404.html` in sync automatically

```js
export default async function handler(request, context) {
  const url = new URL(request.url);
  const pathname = url.pathname;
  const accept = request.headers.get("accept") || "";
  const isHTMLRequest = accept.includes("text/html");

  // Pass through non-HTML and Nuxt internals untouched
  if (!isHTMLRequest || pathname.startsWith('/_nuxt/') || pathname.endsWith('/_payload.json')) {
    return fetch(request);
  }

  const response = await fetch(request);
  if (response.status === 404) {
    url.pathname = "/404.html";
    const fallbackReq = new Request(url.toString(), { method: request.method, headers: request.headers });
    const fallbackResp = await fetch(fallbackReq);
    if (fallbackResp.ok) {
      const html = await fallbackResp.text();
      return new Response(html, {
        status: 404,
        statusText: 'Not Found',
        headers: { 'Content-Type': 'text/html', 'Cache-Control': 'no-store, must-revalidate' }
      });
    }
  }
  return response;
}
```

Key details that make it correct:
- Only intercepts **HTML requests** (`Accept: text/html`); **passes through `/_nuxt/` assets and `_payload.json`** so static assets/data aren't rewritten.
- Returns the fallback with **`status: 404`** (not 200) → SEO-safe.
- **`Cache-Control: no-store, must-revalidate`** on the 404 so a stale 404 isn't cached.

## Option 2 — Switch to SSR mode (no Edge logic needed)

Deploy Nuxt in **SSR mode** instead of fully static. Then **Nuxt resolves routes at runtime** and its **built-in error handling renders the custom error page** with the correct **HTTP 404** natively — **no Edge Function required.** Trade-off: requires a **server runtime** (no longer pure static).

## Recommendation

- **Staying fully static:** Edge-based 404 handling is the **cleanest, most production-safe** option.
- **Open to SSR long-term:** switching to SSR removes the need for Edge logic entirely.

## Generalized guidance (for future similar queries)

- **"Custom 404 not showing on a static (SSG) site — Launch shows its default 404"** → expected: for SSG, **Launch is a static CDN** and serves its **system 404 for non-existent routes before framework logic**. Dropping a `404.html` in the output **isn't auto-served with a 404 status** for arbitrary/nested routes.
- **Fix (SSG):** an **Edge Function** that intercepts upstream 404s and re-serves the framework's `/404.html` with an explicit **404 status** — gate on `Accept: text/html`, pass through framework asset/data paths, and set **no-store** caching. Applies beyond Nuxt (any static framework with a generated 404 page).
- **Fix (alternative):** **SSR mode** gives native framework error handling + correct status with no edge logic, at the cost of a server runtime.
- This is **not** a `launch.json` rewrite problem — rewrites don't preserve the 404 status; use the Edge Function.

## Status

- **Resolved.** Provided the **Edge Function 404 fallback** (recommended for SSG) and the **SSR alternative**; customer confirmed **everything is working** on their end.
