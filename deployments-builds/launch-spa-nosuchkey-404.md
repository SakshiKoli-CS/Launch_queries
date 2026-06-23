QUESTION

A customer deploying a **React starter app** on Launch saw the site build successfully but got a **404 on initial load**, followed by **`NoSuchKey: The specified key does not exist`** on refresh. Build command was `npm run build`, output directory `./build`. A plain Create React App deployed fine; the issue appeared specific to the starter app.

ANSWER

**Cause**

- The React starter is a **single-page application (SPA)**—only static assets are served, with no backend.
- When navigating directly to a deep URL (e.g. `/about-us`), Launch tries to serve a static file at that path. No such file exists, so it returns **`NoSuchKey`**.
- The initial **404** has a separate root cause that requires debugging via browser dev tools / network inspector.

**Workaround**

- Use **`HashRouter`** instead of `BrowserRouter`. Hash-prefixed URLs (`/#/about-us`) keep routing entirely client-side, so the server always serves `index.html` and the app handles the route.

**Platform status**

- Serving `index.html` as a fallback for unmatched routes (the standard SPA fix) was **not supported** at the time of this thread. Support for this out of the box via an **SPA framework preset** was planned for a future release.

**Suggested checks**

- Verify whether the 404 on initial load is a separate issue (e.g. incorrect output directory or missing `index.html`) by inspecting network requests in the browser.
- If the project has since moved to a supported JAMStack framework (Next.js, etc.), server-side routing handles this automatically.
