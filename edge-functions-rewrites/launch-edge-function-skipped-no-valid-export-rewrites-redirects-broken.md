QUESTION

Customer **iO Group / iodigital** (SF case **00057316**) found that **after a deployment** (dev + acceptance), their **Edge Function stopped working**:
- **Images returning 404** — image URL rewriting (handled by the edge function) not working.
- **Redirects not functioning** — expected **308** redirects (e.g. `/about-us.aspx`) not happening.

Both are handled by the **edge function**, so it looked like the function **wasn't triggering**. Customer said **no changes** were made to the edge function or deployment setup; "it worked before 16/4/2026." The function file is under **`functions/`** at the project root.

_Keywords: edge function not triggering, edge function not deployed, rewrites not working, redirects 308 not working, images 404, [proxy].edge.js, does not contain a valid export function, skipping deploying to edge, missing default export, Edge Functions Deployment step._

ANSWER

## Root cause — Edge Function skipped at deploy (no valid export)

- The deployment logs showed, at **Step 5: Edge Functions Deployment**:
  ```
  [proxy].edge.js does not contain a valid export function. Skipping deploying to edge.
  ```
- The Edge Function **was not structured correctly** (no valid **default export**), so Launch **skipped deploying it to the edge**. With the function not deployed, the **rewrites and redirects never run** → images 404, redirects don't fire.
- This is why it "looked like it wasn't triggering" — it genuinely **wasn't deployed**, even though the file was present.

## Fix — valid default export + correct file/location

- The Edge Function must:
  - be **JavaScript**, in a file named **`[proxy].edge.js`**, under the **`/functions`** directory at the **project root**, and
  - have a **valid default export**, e.g.:
    ```js
    export default function handler(request, context) {
      const parsedUrl = new URL(request.url);
      const route = parsedUrl.pathname;
      // ...rewrite/redirect logic...
      return fetch(request);
    }
    ```
- Once the function exports a valid handler, the **Edge Functions Deployment step succeeds** and rewrites/redirects work again. **Confirmed resolved.**
- Docs: Edge Functions (`/launch/edge-functions`). (Edge Functions are **JS only** — author in TS, compile to JS: [edge-functions-typescript.md](edge-functions-typescript.md).)

## Diagnostic — check the deployment log's Edge Functions step

- **Always check the build/deploy log for the "Edge Functions Deployment" step.** A **successful** deploy lists the edge function being deployed; if you instead see **"does not contain a valid export function. Skipping deploying to edge,"** the function is **silently not deployed** — that's the smoking gun.
- "It worked before / we changed nothing" doesn't rule this out — verify the **deployed** state via the logs, not just the file's presence.

## Generalized guidance (for future similar queries)

- **"Edge function rewrites/redirects stopped working after a deploy (404s, redirects not firing)"** → check the deploy log's **Edge Functions Deployment** step. **`[proxy].edge.js does not contain a valid export function → Skipping`** means the function lacks a **valid default export** and wasn't deployed.
- **Setup checklist:** JS file named **`[proxy].edge.js`**, under **`/functions`** at project root, with a **`export default function handler(request, context)`**.
- **The function being present ≠ deployed** — confirm the Edge Functions deploy step succeeded.

## Status

- **Resolved** — the Edge Function lacked a valid default export and was skipped at deploy (`[proxy].edge.js does not contain a valid export function`); correcting the export structure restored the rewrites/redirects. Customer confirmed.
