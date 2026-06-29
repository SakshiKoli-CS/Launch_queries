QUESTION

**First Interstate Bank** reported a deployment error where the application was failing to update a prerendered HTML file at runtime:

```
Failed to update prerender cache for /wordpress [Error: EROFS: read-only file system,
open '/var/task/sourcecode/.next/server/app/wordpress.html']
{ errno: -30, code: 'EROFS', syscall: 'open',
  path: '/var/task/sourcecode/.next/server/app/wordpress.html' }
```

The question was whether Launch was blocking writes to the deployed filesystem.

ANSWER

**Expected behavior — the filesystem is read-only by design**

On Launch, the deployed filesystem under `/var/task/` is **read-only after deployment**. Any attempt to write or update prerendered files at runtime (such as ISR-style prerender cache updates writing to `.next/server/app/*.html`) will fail with `EROFS: read-only file system`.

This is a platform constraint, not a bug. Serverless/container runtimes do not allow modifying deployed build output.

**How this differs from `.next/cache` ENOENT errors**

| | This error | `.next/cache` ENOENT |
|---|---|---|
| Error code | `EROFS` (writing to read-only path) | `ENOENT` (can't create directory) |
| Path | `.next/server/app/<page>.html` (prerender output) | `.next/cache/` (Next.js cache dir) |
| Cause | ISR / prerender cache update at runtime | Next.js cache directory not writable |
| Fix | Use CDN-level revalidation instead | Redirect cache to `/tmp` via `cacheHandlers` |

See: [`.next/cache` ENOENT → blank pages](launch-nextjs-cache-enoent-blank-pages-readonly-fs.md)

**Recommended fix — CDN-level cache revalidation**

Rather than writing prerender files to disk, use Launch's CDN revalidation mechanisms to refresh content when it changes:

1. **Launch Public API** — trigger cache invalidation for specific paths, domains, or cache tags on publish/update
2. **Cache tags** — associate pages with cache tags and selectively revalidate when content changes
3. **Automate** — configure a workflow to trigger CDN cache revalidation automatically when CMS entries are published

These approaches refresh content at the CDN layer without touching the filesystem, which is the correct pattern for Launch deployments.

**Resolution status**

Workaround communicated; no follow-up response from customer confirming implementation.
