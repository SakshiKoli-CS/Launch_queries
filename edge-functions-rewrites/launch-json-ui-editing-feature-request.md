QUESTION

A partner asked whether it is on the roadmap to create and edit the `launch.json` file (which controls Edge URL Rewrites and redirects) through the Launch UI, similar to how Environment Variables are managed. The use case: marketers want to create vanity URLs and redirects without going through a developer.

ANSWER

**Current status — not on roadmap (as of March 2026)**

Editing `launch.json` via the Launch UI is not currently supported or on the roadmap. The standard answer for marketer-managed redirects is to implement a **redirect/rewrite content type** in Contentstack and have the application read from it — but this is seen as additional implementation effort.

**Why it hasn't been prioritised**

- Moving redirect/rewrite config out of version control negates one of the key advantages of keeping `launch.json` in the codebase (auditability, rollback, peer review)
- The feature request had not previously surfaced from customers at the time of this thread

**Proposed approach (under consideration)**

Take a **union** of redirects/rewrites from:
1. The codebase (`launch.json`) — read-only in the UI
2. The Launch UI — editable by non-developers

This would allow marketers to manage their own redirects while keeping developer-owned rewrites locked in version control. The concept has been added to the product board for evaluation.

**Current workaround for marketer-managed redirects**

- Create a **redirect content type** in Contentstack with fields for source path, destination URL, and redirect type
- The application fetches and applies these at runtime or build time
- More implementation effort upfront but gives full CMS-driven control with no developer involvement for day-to-day redirect management

**Resolution**

Feature concept added to the product board. No ETA; evaluate feasibility pending.
