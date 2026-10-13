---
type: "practice"
difficulty: "Deep"
---

# How would you cache route-level data in a Next.js App Router app using tag-based revalidation so a CMS update refreshes only the affected pages?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Stale While Revalidate](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Stale_While_Revalidate.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

In a Next.js App Router application, implementing tag-based revalidation with caching is crucial to efficiently handle updates in a content management system (CMS). By leveraging tags and incremental static regeneration (ISR), we can ensure that only the affected routes are updated when changes occur in the CMS. This approach balances performance and update granularity.

#### Triaging the Question

The interviewer is probing your understanding of Next.js App Router, caching strategies, tag-based revalidation, and how to integrate these concepts with a content management system (CMS). Key assumptions include familiarity with Next.js features, understanding static generation vs dynamic rendering, and knowledge of caching mechanisms.

Step-by-Step Solution:

Understanding App Router and Static Generation:

App Router in Next.js is designed for serverless architecture and supports both static and incremental static regeneration (ISR).
Static Generation: Routes are pre-rendered at build time, which can be slow but results in fast page loads.
Incremental Static Regeneration (ISR): Pages are regenerated on-demand when a request hits the server, providing a balance between speed and freshness.

Implementing Tag-Based Revalidation:

We will use tags to tag our routes with metadata that can trigger revalidation.
When a CMS update occurs, we programmatically invalidate the specific tags associated with those pages.

Setting Up ISR in Next.js:

In your route handler or pages directory, you can set up ISR by specifying the revalidations option in the routing file.

Tagging and Validating Routes:

Each CMS page can be tagged with metadata that identifies it (e.g., a unique ID).
When content changes in the CMS, you update this metadata.
In your Next.js app, listen for these updates and invalidate the corresponding tags using getServerSideProps or custom hooks.

#### Example
Implementation:

JAVASCRIPT
// pages/[id].js
import { useRevalidateTag } from 'next-revalidate-tag';

#### cons
t Page = ({ content }) => {
  return (
    <div>
      {/* Render content */}
    </div>
  );
};

export async function getServerSideProps(context) {

#### cons
t { params, revalidateTags } = context;

#### cons
t id = params.id;

  // Fetch data from CMS

#### cons
t content = await fetchContentFromCMS(id);

  // Invalidate the tag if needed (e.g., when content changes)
  revalidateTags(`page-${id}`);

  return {
    props: { content },
  };
}

export default Page;

Handling Invalidation in CMS:

When a CMS update occurs, you can programmatically invalidate the relevant tags using an API or script.
JAVASCRIPT
// Example API endpoint for invalidating tags
import { revalidateTags } from 'next-revalidate-tag';

export async function POST(req) {

#### cons
t { id } = await req.json();
  revalidateTags(`page-${id}`);
  return new Response('OK', { status: 200 });
}

Ensuring Granularity and Performance:

By tagging and invalidating only the specific pages that need to be updated, we minimize unnecessary regeneration.
This approach ensures that only affected routes are re-rendered, improving performance and reducing server load.

#### Closing Summary

Tag-based revalidation in Next.js App Router allows for efficient management of CMS updates by ensuring only the relevant pages are re-generated. By leveraging ISR and carefully managing tags, you can achieve a balance between content freshness and performance. Common pitfalls include not tagging all routes appropriately or failing to invalidate tags when necessary.

#### Common Mistakes/Follow-Ups
Not Using Tags Properly: Failing to correctly associate tags with routes can lead to unnecessary regeneration.
Improper Invalidation Timing: Invalidating too frequently can impact server load, while infrequent invalidations may cause stale content.
Ignoring Edge Cases: Not handling edge cases such as simultaneous updates or race conditions can result in inconsistencies.

**Comparison:**

```
Yes
No
Page Route
Is Tagged?
Revalidate Tag
Continue Rendering
```

**Code:**

```

// pages/[id].js import { useRevalidateTag } from 'next-revalidate-tag';

const Page = ({ content }) => { return ( <div> {/* Render content */} </div> ); };

export async function getServerSideProps(context) { const { params, revalidateTags } = context; const id = params.id;

// Fetch data from CMS const content = await fetchContentFromCMS(id);

// Invalidate the tag if needed (e.g., when content changes) revalidateTags(page-${id});

return { props: { content }, }; }

export default Page;

CODE
Copy

```javascript
// Example API endpoint for invalidating tags
import { revalidateTags } from 'next-revalidate-tag';

export async function POST(req) {
  const { id } = await req.json();
  revalidateTags(`page-${id}`);
  return new Response('OK', { status: 200 });
}
```

**Example input / output:**

| Input (CMS Update) | Output (Page Revalidation) |
|--------------------|----------------------------|
| CMS updates page 1's title to "Welcome Home" | `page-1` is revalidated and rendered with the new title. |
| CMS updates page 2's content | `page-2` is revalidated and rendered with the updated content. |

**Data:**

```json
// Example CMS data JSON schema
{
  "pages": [
    {
      "id": 1,
      "title": "Home",
      "content": "Welcome to our website!"
    },
    {
      "id": 2,
      "title": "About Us",
      "content": "We are a content management system."
    }
  ]
}
```

**Flow:**

1. +-------------------+
2. | CMS Updates Page 1|
3. +--------+---------+
4. |
5. v
6. +-------------------+
7. | Invalidate Tag (page-1)|
8. +----+-----+----------+
9. |
10. v
11. +-----------------------+
12. | ISR Triggers Revalidation of page-1|
13. +-------------------------------+

