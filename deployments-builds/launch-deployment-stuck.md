QUESTION

A customer's Launch deployment appeared to hang for 12+ minutes with no progress. Their build command included `SET PORT=8080 && node ./server/index.js`, intending to run a **CRA frontend + Node/Express backend API** (client and server in separate folders). They asked whether deployments can be cancelled.

ANSWER

**Why the deployment hung**

- The build command contained `node ./server/index.js`, which **starts a long-running Express server process**. Launch interprets the build command as a build step—not a runtime command—so the process runs indefinitely, causing the deployment to appear stuck.
- **Fix:** remove the `node ./server/index.js` portion entirely. Launch handles serving the site after the build completes; starting a server in the build command is not required and will block the deployment.

**Deployment timeout**

- Hung deployments will **automatically timeout and stop after 60 minutes**. There is no manual cancel option.

**Output directory**

- For a CRA project inside a `client/` subfolder, set **Output Directory** to `./client/build`.

**Running a standalone Express backend on Launch**

- **Not supported.** Launch is focused on JAMStack frameworks (Next.js, Gatsby, 11ty, SvelteKit, etc.). Running a persistent Express server would require a full PaaS solution (like Heroku).
- **Alternative for a static frontend + backend API pattern:** use **Launch Cloud Functions** to handle API endpoints instead of a standalone Express server.

**Suggested checks**

- Ensure the build command only runs the **build step** (e.g. `npm run build`), not a server start command.
- Confirm Output Directory points to the folder where the static build output is generated.
