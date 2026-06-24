QUESTION

**Launch Edge Functions are JavaScript-only** per the docs, but the customer's codebase is **TypeScript** throughout. Is **TypeScript support planned**? What's the **recommended workaround**? (Also asked how Edge Functions relate to Cloud/serverless functions.)

_Keywords: Edge Functions TypeScript, JavaScript only, compile TS to JS, tsconfig, functions directory, WinterCG, edge runtime vs Node.js, lighter runtime, cloud functions comparison._

ANSWER

## TypeScript — author in TS, compile to JS (no native TS execution)

- Launch Edge Functions **support only JavaScript** and do **not natively execute TypeScript**.
- **Workaround:** write Edge Functions in **TypeScript** and **compile to JavaScript** as part of the Launch **build process**; place the compiled JS in the **`functions/`** directory. (This is the same pattern Next.js etc. use — author in TS, ship JS.)
- Starting `tsconfig.json`:
  ```json
  {
    "compilerOptions": {
      "target": "ES2022",
      "module": "ES2022",
      "moduleResolution": "bundler",
      "outDir": "./functions",
      "rootDir": "./src/edge",
      "strict": true,
      "esModuleInterop": true,
      "skipLibCheck": true,
      "forceConsistentCasingInFileNames": true
    },
    "include": ["src/edge/**/*"]
  }
  ```
- So the **rest of the codebase being TypeScript doesn't matter** — only the **artifact** Launch runs needs to be JS. (See also the existing entry: [edge-functions-typescript.md](edge-functions-typescript.md).)

## Edge Functions vs Cloud/serverless functions — runtime differences

- Edge Functions are **similar to Cloud Functions** but the **edge runtime is lighter** than Node.js and **may not support all Node.js features**.
- Edge Functions are **WinterCG-compliant** (Minimum Common Web Platform API): https://min-common-api.proposal.wintertc.org/ — so target **web-standard / WinterCG APIs** in edge code, not Node-specific APIs that may be unavailable at the edge.

## Generalized guidance (for future similar queries)

- **"Can we use TypeScript for Edge Functions?"** → Not natively (JS only); **compile TS → JS** in the build and output to **`functions/`**. The broader codebase can stay TS.
- **"What can the edge runtime do?"** → It's **lighter than Node.js** and **WinterCG-compliant** — don't assume full Node API availability in Edge Functions; use web-standard APIs. (Cloud Functions are the Node.js-based option for heavier server logic.)

## Status

- Answered: **JS-only**, compile TS→JS into `functions/` (tsconfig provided); edge runtime is **lighter than Node + WinterCG-compliant**. (No date given for native TS support.)
