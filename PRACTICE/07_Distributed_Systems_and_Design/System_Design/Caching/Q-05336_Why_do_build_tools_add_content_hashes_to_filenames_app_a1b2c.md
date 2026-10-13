---
type: "practice"
difficulty: "Intermediate"
---

# Why do build tools add content hashes to filenames (app.a1b2c3.js), and how does that interact with long-lived caching?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Web and Browser Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Web_and_Browser_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Content hashing in filenames, like app.a1b2c3.js, is a common practice used by build tools to ensure cache freshness and improve performance. Essentially, it allows browsers to efficiently update their caches when the underlying source files change without needing to check for differences in content. This technique helps avoid unnecessary re-fetches of assets that haven't changed.

To understand why this is important, let's break down what happens when a browser requests an asset from a server:

Initial Request: The browser fetches app.js and caches it.
Subsequent Requests: If the underlying JavaScript file changes but remains named app.js, the browser will not know if the content has changed or if it can use its cached version.

Content hashing solves this by embedding a unique hash of the file's contents into the filename. When the file changes, so does the hash, ensuring that a new filename is generated and the old one invalidated in the cache.

#### Triage
What the Question Is Asking

The core question here involves understanding:

Why content hashes are used: To ensure accurate caching behavior.
How these hashes interact with long-lived caching mechanisms like browser caches.

#### Assumptions
and First Principles Solution

Assume we're working with a typical web development setup where assets are cached by both the client (browser) and server. The goal is to balance between serving fresh content and avoiding unnecessary network requests.

Step-by-Step Reasoning:

File Content vs Filename:

A filename alone doesn't tell us if the file's contents have changed.
For example, app.js could refer to either an older version or a new one, depending on its content.

Content Hashes in Filenames:

By including a hash of the file’s contents (e.g., a1b2c3) in the filename (app.a1b2c3.js), we ensure that the filename changes when the content changes.
The browser can then use this hashed filename to determine if it needs to fetch the updated version.

Cache Invalidation:

When a file is updated, its hash also changes, leading to a new filename (e.g., app.b4d5f6.js).
This forces the browser to re-fetch and potentially update its cache with the new content.
Conversely, if no changes occur, the same hashed filename remains valid, reducing unnecessary requests.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Content hashing is a robust strategy for ensuring that browsers can efficiently manage their caches. By embedding file hashes in filenames, build tools guarantee that only updated assets trigger fresh downloads, optimizing performance and minimizing network overhead.

#### Common Mistakes
Failing to Generate Unique Hashes: If the same content generates the same hash, it won't work as intended.
Over-Reliance on Caching Mechanisms: Assuming that all clients (browsers) will respect and utilize long-lived cache policies consistently might lead to issues.

#### Follow-Ups
Discuss edge cases like file deletions or renames without changes in content.
Explore how this impacts server-side caching strategies.

#### Cons
ider the impact of frequent updates on performance metrics.

**Comparison:**

```
Scenario	Cache Behavior	Outcome
File content unchanged	No new request, use cached version	Efficiently serves stale content
File content updated	New filename generated	Forces browser to re-fetch and update
```

**Code:**

```javascript
// Example of a hashed filename generation function (pseudo-code)
function generateHash(fileContent) {
  // Use a hashing algorithm like MD5 or SHA1 to generate a unique hash for the file content
  return crypto.createHash('md5').update(fileContent).digest('hex');
}

const originalContent = "console.log('Hello, world!')";
const hashedFilename = `app.${generateHash(originalContent)}.js`;
```

**Example input / output:**

```text
Input:
- Initial request for `app.js` (cached)
- Updated content in `app.js`

Output:
- Rebuilt asset with new hash, e.g., `app.b4d5f6.js`
- Browser receives the new filename and re-fetches if necessary to update its cache.
```

**Data:**

```json
{
  "filename": "app.a1b2c3.js",
  "contentHash": "a1b2c3"
}
```

**Flow:**

1. Initial Request
2. Cache Hit -> Serve CachedVersion
3. File Updated
4. Rebuild with New HashedFilename
5. Browser Checks New Filename
6. Browser Determines ContentChanged
7. Serve New Asset, UpdateCache
8. Content Unchanged
9. Browser Uses Stale Cache

