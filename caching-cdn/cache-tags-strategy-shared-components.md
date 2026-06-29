QUESTION

A prospect asked how to keep a Next.js site on Launch up-to-date without a full deployment when content shared across multiple pages changes. They had been planning to use Next.js's `revalidateTag()` to tag shared components and let Next.js know which pages to revalidate — but the Launch docs note that `revalidateTag()` does not work on Launch. They asked:

1. Does Launch only support per-URL invalidation?
2. What is the recommended alternative for invalidating a shared component that appears on many pages?
3. How should cache tags be compiled — what is the best tagging strategy?

ANSWER

**Clarification — two different "cache tags" concepts**

Next.js's built-in `revalidateTag()` (App Router) is **not supported on Launch** — this is the in-process Next.js primitive that works only in Vercel's own runtime. See also: [ISR and revalidation cap](launch-isr-support-cache-revalidation-limit-cap.md).

Launch has its **own CDN-level cache tag system**, which is independent of Next.js and works with any framework. This is the recommended alternative. Tags are attached to CDN responses via a `Cache-Tags` response header; Launch's revalidation API or Automate can then purge all cached URLs sharing a given tag in a single call.

**Options for triggering revalidation**

1. **Launch Public API** — call the Revalidate Cache endpoint with specific paths, domains, or cache tags. [Docs: Public APIs – Revalidate Cache]
2. **Automate** — use the `revalidate-cache-cdn` action in an Automate workflow triggered by a Contentstack publish event. [Docs: Automate – Revalidate Cache]
3. **Cache tag invalidation** — assign tags to shared components or content entries; a single invalidation call clears all pages that carry that tag, without a full redeployment.

**Cache tag strategies**

Three common approaches, often combined:

| Strategy | How it works | Trade-off |
|---|---|---|
| **Entry UID** | Tag each page with the UIDs of the main entry and all referenced entries | Most precise invalidation; larger `Cache-Tags` header on pages with many references |
| **Content type** | Tag by content type (`page`, `article`, `author`) | Simpler; may over-invalidate (all pages of that type are cleared when any entry changes) |
| **Component** | Tag by reusable component (`component-hero`, `component-navigation`) | Good balance — a single component change clears only pages using that component |

In practice, many implementations combine all three: `Cache-Tags: <entry_uid>, <content_type>, <component_name>`.

**Compiling UID-based cache tags — recursive extraction pattern**

For UID-based tagging, the page must extract UIDs from the main entry and all nested references. A recursive helper:

```ts
// utils/contentstack-helper.ts
export function extractCacheTags(data: any, tags: Set<string> = new Set()): string[] {
  if (!data || typeof data !== 'object') return Array.from(tags);
  if (data.uid) tags.add(data.uid);
  if (Array.isArray(data)) {
    data.forEach(item => extractCacheTags(item, tags));
  } else {
    Object.values(data).forEach(value => {
      if (value && typeof value === 'object') extractCacheTags(value, tags);
    });
  }
  return Array.from(tags);
}
```

Setting the `Cache-Tags` header on the response (Next.js Route Handler example):

```ts
// app/api/page-data/route.ts
import { NextResponse } from 'next/server';
import Stack from '@/lib/contentstack';
import { extractCacheTags } from '@/utils/contentstack-helper';

export async function GET(request: Request) {
  const entry = await Stack.ContentType('page')
    .Entry('YOUR_ENTRY_UID')
    .includeReference(['related_articles', 'author', 'blocks.reference_field'])
    .fetch();

  const entryData = entry.toJSON();
  const tags = extractCacheTags(entryData);

  return NextResponse.json(entryData, {
    headers: {
      'Cache-Tags': tags.join(','),         // e.g. "blt123,blt456,blt789"
      'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=30',
    },
  });
}
```

**Practical notes**

- **Reference depth:** the SDK defaults to a fixed depth. Use `.includeReference(['ref1', 'ref1.ref2'])` for deeply nested references, or the helper above will miss nested UIDs.
- **Header size:** CDNs typically cap response headers at 8–16 KB. If a page has hundreds of references, filter tags to the most critical entries rather than including every UID.
- **App Router Page components** cannot set response headers directly. Set the `Cache-Tags` header in a **Middleware** or **Route Handler**, or use the component-/content-type-based approach (which can be applied more broadly without per-page header injection).

**Resolution**

Prospect confirmed the Launch cache tag approach (using Contentstack entry UIDs + Automate invalidation) covers their shared-component invalidation use case without a full redeployment.
