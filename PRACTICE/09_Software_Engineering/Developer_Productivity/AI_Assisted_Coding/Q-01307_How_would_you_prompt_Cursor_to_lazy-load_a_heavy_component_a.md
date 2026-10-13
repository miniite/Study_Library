---
type: "practice"
difficulty: "Intermediate"
---

# How would you prompt Cursor to lazy-load a heavy component and confirm the bundle actually shrank?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Agentic Coding with Cursor](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Agentic_Coding_with_Cursor.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle the scenario where we want to prompt Cursor (an AI-assisted coding tool like GitHub Copilot) to lazy-load a heavy component while ensuring that the code bundle size indeed shrinks, we need to understand both the technical aspects of lazy-loading and how to effectively communicate our requirements to the AI. Here’s a step-by-step approach:

#### Triaging the Question

The key here is twofold:

Lazy-Loading: We want to defer the loading of heavy components until they are actually needed, which can improve initial load times.
Bundle Size Confirmation: We need to ensure that the lazy-loading process actually results in a smaller code bundle.

#### Assumptions
You have some understanding of how lazy-loading works with modules in JavaScript (e.g., using React.lazy or dynamic imports).
The project involves using a tool like Cursor for AI-assisted coding.
You are familiar with bundlers like Webpack, which can be used to measure and optimize bundle sizes.

#### First Principles Solution
Define the Lazy-Loading Logic: Use dynamic imports in your codebase where heavy components are only imported when needed.
Prompt Cursor: Provide clear instructions on how to implement lazy-loading using specific syntax or patterns.
Measure Bundle Size: Before and after implementing the changes, measure the bundle size to confirm that it has indeed shrunk.

#### Implementation
Steps

Identify Heavy Components:

Identify components that are heavy in terms of size or complexity (e.g., complex UIs with many dependencies).

Implement Lazy-Loading:

JAVASCRIPT
import React, { Suspense } from 'react';

#### cons
tHeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <div>
      {/* Other components */}
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </div>
  );
}

Prompt Cursor:

In the code editor, you might prompt Cursor to use React.lazy and dynamic imports for heavy components.
JAVASCRIPT
// Prompt Cursor: "Use React.lazy and dynamic imports to lazy-load HeavyComponent"

#### cons
tHeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <div>
      {/* Other components */}
      <Suspense fallback={<div>Loading...</div>}>
        <LazyHeavyComponent />
      </Suspense>
    </div>
  );
}

Measure Bundle Size:

Use tools like Webpack’s built-in size statistics or third-party tools like bundlephobia to measure the bundle sizes.
BASH
npx webpack --profile --json > stats.json
node ./report.js # generates a report with bundle sizes

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Forgetting to import the heavy component dynamically, which can lead to unnecessary code inclusion.

#### Follow-Up
Question: "How would you handle performance issues if lazy-loading still leads to noticeable delays?"
Additional Consideration: Ensure that the lazy-loaded components are efficiently rendered and do not cause additional overhead.

**Comparison:**

```
Component	Before Lazy-Loading (bytes)	After Lazy-Loading (bytes)
HeavyComponent	50,000	-
App	12,000	3,000
Total Bundle Size	62,000	45,000
```

**Code:**

```javascript
import React, { Suspense } from 'react';
constHeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <div>
      {/* Other components */}
      <Suspense fallback={<div>Loading...</div>}>
        <LazyHeavyComponent />
      </Suspense>
    </div>
  );
}
```

**Example input / output:**

```text
Before lazy-loading:
- Total bundle size: 62,000 bytes

After implementing lazy-loading:
- `HeavyComponent` is only loaded when needed.
- New total bundle size: 45,000 bytes
```

**Data:**

```json
{
  "stats": [
    {"name": "HeavyComponent", "size": 50000},
    {"name": "App", "size": 12000}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Identify Heavy Component
4. Is Dynamic Import Needed?
5. Implement Lazy-Loading withReact.lazy
6. No Changes Required
7. Measure Bundle Size
8. Confirm Bundle Shrinkage

