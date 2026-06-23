QUESTION

A customer (Valtech) asked whether **Launch cloud functions are incompatible with monorepos and Next.js**, based on a limitation noted in the (pre-GA) documentation.

ANSWER

**Cloud functions + monorepo**

- **Launch cloud functions are not supported in monorepos.** This is a known limitation, as documented.

**Next.js API endpoints are the alternative**

- If the project uses **Next.js**, there is no need to use Launch cloud functions. **Next.js API endpoints (`/api` routes) provide equivalent functionality** and **do work** on Launch, including in a monorepo context.

**Edge functions**

- **Edge functions** (distributed/CDN-layer serverless functions) were **not yet supported** at the time of this thread and were planned for a later release.

**Suggested checks**

- If a monorepo project needs server-side logic, confirm it is using **Next.js API routes** rather than Launch cloud functions.
- Edge function availability should be verified against current Launch release notes, as this was a pre-GA limitation.
