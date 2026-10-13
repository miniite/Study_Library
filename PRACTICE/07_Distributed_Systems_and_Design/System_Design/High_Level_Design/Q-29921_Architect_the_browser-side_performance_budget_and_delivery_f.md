---
type: "practice"
difficulty: "Deep"
---

# Architect the browser-side performance budget and delivery for a data-heavy dashboard SPA that must stay under a strict LCP/INP even with 200 API-driven widgets — how do you split code stream data and prioritize above-the-fold without a waterfall?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Micro-frontends](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Micro-frontends.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this high-level design (HLD) for a browser-side performance budget and delivery of a data-heavy dashboard Single Page Application (SPA), we need to ensure that even with 200 API-driven widgets, the Largest Contentful Paint (LCP) and Interactive Time (INP) remain within strict limits. The key challenge is to prioritize above-the-fold content while streamlining code and data loading without causing a waterfall effect.

#### Triaging the Question

The core of this problem involves optimizing the initial load time and ensuring interactive performance by carefully managing how and when resources are loaded. Specifically, we need to:

Prioritize Above-the-Fold Content: Ensure that the most critical content loads first.
Asynchronous Loading: Minimize blocking of the main thread by deferring non-critical resources.
Code Splitting: Break down large JavaScript bundles into smaller chunks to reduce initial load time.

#### First Principles Solution

To achieve this, we will employ a combination of code splitting, lazy loading, and optimizing network requests. Here’s a step-by-step breakdown:

Above-the-Fold Optimization:

Identify the critical rendering path (CRP) elements.
Ensure that these elements are loaded first to meet LCP/INP requirements.

Code Splitting:

Use dynamic imports for non-critical JavaScript files.
Split large JavaScript bundles into smaller, more manageable chunks using tools like Webpack's splitChunks optimization.

Lazy Loading:

Implement lazy loading for images and other media to ensure they are only loaded when needed.
Use Intersection Observer API or similar techniques to defer loading of non-critical widgets until they come into the viewport.

Async Fetching:

For API-driven data, use asynchronous fetching with fetch or axios.
Implement a fallback mechanism to provide placeholder content while actual data is being loaded.

#### Implementation
Details

Above-the-Fold Content:

HTML
<!-- Example above-the-fold HTML -->
<div id="above-the-fold">
  <!-- Critical elements like charts, main content here -->
</div>

Code Splitting:

Use Webpack or Rollup to split the code into smaller chunks.
JAVASCRIPT
// webpack.config.js example
optimization: {
  splitChunks: {
    cacheGroups: {
      vendor: {
        test: /[\\/]node_modules[\\/]/,
        name: 'vendors',
        chunks: 'all'
      }
    }
  }
}

Lazy Loading:

JAVASCRIPT
// Example of lazy loading an image
import { lazy } from 'react';

#### cons
t LazyImage = lazy(() => import('./LazyImage'));

function App() {
  return (
    <div>
      {/* Critical content */}
      <LazyImage />
    </div>
  );
}

Async Fetching:

JAVASCRIPT
// Example of async fetching for API data

#### cons
t fetchData = async (url) => {
  try {

#### cons
t response = await fetch(url);
    if (!response.ok) throw new Error('Network response was not ok');
    return await response.json();
  } catch (error) {

#### cons
ole.error('Failed to fetch data:', error);
  }
};

fetchData('/api/data')
  .then(data => console.log(data))
  .catch(err => console.error(err));

#### Closing Summary

By carefully managing above-the-fold content, implementing code splitting and lazy loading, and using asynchronous fetching, we can ensure that the data-heavy dashboard remains performant within strict LCP/INP limits. The key is to prioritize critical elements first and defer non-critical resources until they are needed.

#### Common Mistakes/Follow-Ups
Not Prioritizing Above-the-Fold Correctly: Failing to identify and load only the most critical content upfront can lead to poor initial performance.
Blocking Main Thread with Blocking Resources: Overlooking blocking scripts or stylesheets that could delay rendering.
Inefficient Code Splitting: Poorly configured code splitting can result in larger initial bundles, defeating the purpose.

**Comparison:**

```
Code Splitting Strategy	Description	Example
Dynamic Imports	Load modules only when needed	import('./LazyWidget').then(widget => widget.render())
Tree Shaking	Remove unused code from bundles	Optimized by Webpack/ Rollup
```

**Code:**

```javascript
// Example of dynamic import and lazy loading
const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <div>
      {/* Critical content */}
      <LazyComponent />
    </div>
  );
}
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `fetchData('/api/data')` | Fetched data and logs it to the console. |

**Data:**

```json
{
  "data": [
    {"id": 1, "name": "Widget A", "content": "Loading..."},
    {"id": 2, "name": "Widget B", "content": "Loading..."}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is content above-the-fold?
5. Load critical content first
6. Lazy load non-criticalresources
7. Check LCP/INP metrics
8. Monitor async loadingperformance
9. Optimize further if needed
10. Fallback to placeholders ifnecessary
11. End

