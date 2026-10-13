---
type: "practice"
difficulty: "Deep"
---

# You maintain an in-memory LRU cache in a Node service and prod shows memory growing unbounded despite a fixed capacity; how do you root-cause whether the eviction map is actually releasing references and fix it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ LRU Cache Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/LRU_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The issue likely lies within the implementation of your LRU (Least Recently Used) cache, particularly in how the eviction mechanism releases memory references. We need to debug the code to ensure that when an entry is evicted, its reference count indeed drops, and we must also consider potential edge cases like circular references.

Triaging the Problem:

The question revolves around diagnosing a memory leak issue in an LRU cache implementation within a Node.js service. The primary goal is to determine if the eviction process correctly removes entries from both the cache data structure and the underlying object graph, ensuring that reference counts are properly managed.

#### Assumptions
You have implemented an LRU cache with a fixed capacity.
There might be circular references or other issues preventing garbage collection.

#### First-Principles Solution

Review the Eviction Mechanism:

Ensure that when you evict an entry, its reference count is correctly reduced to zero.
Verify if there are any lingering circular references that could prevent garbage collection.

Instrumentation and Debugging:

Use Node.js’s built-in process.memoryUsage() to monitor memory usage over time.
Implement logging for cache operations (insertions, evictions) to trace the lifecycle of each entry.
Utilize tools like heapdump or node-inspector to visualize and inspect the object graph.

Circular References Check:

Write a function to detect circular references in your objects using techniques like traversing the object graph with a set or map to keep track of visited nodes.

Memory Profiling and Leak Detection:

Use node --trace-gc to get detailed garbage collection logs.
Implement custom memory profiling functions to periodically check the size of specific objects.

Code Review and Refactoring:

Ensure that when an entry is removed from the cache, all related references are also removed or released.
Simplify complex data structures and ensure that any shared state is managed properly.
Detailed Steps:

Logging Cache Operations:

JAVASCRIPT
// Example logging function for LRU cache operations

#### cons
t logCacheOperation = (op) => {

#### cons
ole.log(`[Cache] ${op} operation at time:`, new Date().toISOString());
};

class LRUCache {

#### cons
tructor(capacity) {
    this.capacity = capacity;
    this.cacheMap = new Map();
    this.accessOrder = [];
    logCacheOperation('Constructor');
  }

  get(key) {

#### cons
t value = this.cacheMap.get(key);
    if (value !== undefined) {
      this._updateAccessOrder(key); // Update access order
      return value;
    }
    logCacheOperation(`Miss ${key}`);
    return null;
  }

  set(key, value) {
    if (this.cacheMap.has(key)) {
      this.cacheMap.set(key, value);
      this._updateAccessOrder(key); // Update access order
      return;
    }

#### cons
t evictKey = this._evict();
    if (evictKey !== undefined) {
      logCacheOperation(`Evict ${evictKey}`);
    }

    this.cacheMap.set(key, value);
    this.accessOrder.push(key);
  }

  _updateAccessOrder(key) {
    // Update access order

#### cons
t index = this.accessOrder.indexOf(key);
    if (index !== -1) {
      this.accessOrder.splice(index, 1);
      this.accessOrder.push(key);
      logCacheOperation(`Updated ${key} in access order`);
    }
  }

  _evict() {
    // Eviction logic

#### cons
t key = this.accessOrder.shift();
    if (key !== undefined) {
      this.cacheMap.delete(key);
      return key;
    }
    return undefined;
  }
}

function main() {

#### cons
t cache = new LRUCache(3);

  cache.set('a', 1);
  cache.set('b', 2);
  cache.set('c', 3);
  logCacheOperation(`Current State: ${cache.cacheMap}`);

#### cons
ole.log(cache.get('b')); // 2
  logCacheOperation(`Current State: ${cache.cacheMap}`);

  cache.set('d', 4); // Evicts 'a'
  logCacheOperation(`Current State: ${cache.cacheMap}`);
}

main();

Detecting Circular References:

JAVASCRIPT
function hasCircularReference(obj) {

#### cons
t visited = new WeakSet();

  function detectCircularReferences(object, parentObject) {
    if (visited.has(object)) return false;
    visited.add(object);

    for (const key in object) {
      if (object.hasOwnProperty(key)) {

#### cons
t value = object[key];
        if (value === parentObject || (typeof value === 'object' && detectCircularReferences(value, object))) {
          return true;
        }
      }
    }
    return false;
  }

  return detectCircularReferences(obj);
}

// Example usage

#### cons
t obj1 = { a: 1 };

#### cons
t obj2 = { b: 2 };
obj1.c = obj2;
obj2.d = obj1;

#### cons
ole.log(hasCircularReference(obj1)); // true

Memory Profiling with heapdump:

JAVASCRIPT
// Install heapdump using npm: npm install --save-dev heapdump

#### cons
t { createHeapDump } = require('heapdump');

setImmediate(() => {
  createHeapDump({path: '/tmp/my.heap'})
    .then(path => console.log(`Created dump at ${path}`))
    .catch(err => console.error(err));
});

#### Closing Summary

By thoroughly instrumenting the cache operations, using logging and memory profiling tools, and ensuring that circular references are handled correctly, we can effectively root-cause and fix the issue of unbounded memory growth in the LRU cache. Common mistakes include not properly managing reference counts or failing to detect and break circular references.

#### Common Mistakes/Follow-Ups
Not Logging Cache Operations: Lack of logging makes it hard to trace which operations are causing issues.
Circular References Not Handled Properly: Circular references can prevent garbage collection, leading to memory leaks.
Inadequate Memory Profiling: Without proper profiling tools and techniques, it's difficult to identify the root cause of memory leaks.

**Code:**

```

const logCacheOperation = (op) => { console.log([Cache] ${op} operation at time:, new Date().toISOString()); };

class LRUCache { constructor(capacity) { this.capacity = capacity; this.cacheMap = new Map(); this.accessOrder = []; logCacheOperation('Constructor'); }

get(key) { const value = this.cacheMap.get(key); if (value !== undefined) { this._updateAccessOrder(key); // Update access order return value; } logCacheOperation(Miss ${key}); return null; }

set(key, value) { if (this.cacheMap.has(key)) { this.cacheMap.set(key, value); this._updateAccessOrder(key); // Update access order return; }

CODE
Copy
const evictKey = this._evict();
if (evictKey !== undefined) {
  logCacheOperation(`Evict ${evictKey}`);
}

this.cacheMap.set(key, value);
this.accessOrder.push(key);

}

_updateAccessOrder(key) { // Update access order const index = this.accessOrder.indexOf(key); if (index !== -1) { this.accessOrder.splice(index, 1); this.accessOrder.push(key); logCacheOperation(Updated ${key} in access order); } }

_evict() { // Eviction logic const key = this.accessOrder.shift(); if (key !== undefined) { this.cacheMap.delete(key); return key; } return undefined; } }

function main() { const cache = new LRUCache(3);

cache.set('a', 1); cache.set('b', 2); cache.set('c', 3); logCacheOperation(Current State: ${cache.cacheMap});

console.log(cache.get('b')); // 2 logCacheOperation(Current State: ${cache.cacheMap});

cache.set('d', 4); // Evicts 'a' logCacheOperation(Current State: ${cache.cacheMap}); }

main();

CODE
Copy

```javascript
function hasCircularReference(obj) {
  const visited = new WeakSet();

  function detectCircularReferences(object, parentObject) {
    if (visited.has(object)) return false;
    visited.add(object);

    for (const key in object) {
      if (object.hasOwnProperty(key)) {
        const value = object[key];
        if (value === parentObject || (typeof value === 'object' && detectCircularReferences(value, object))) {
          return true;
        }
      }
    }
    return false;
  }

  return detectCircularReferences(obj);
}

const obj1 = { a: 1 };
const obj2 = { b: 2 };
obj1.c = obj2;
obj2.d = obj1;

console.log(hasCircularReference(obj1)); // true
JAVASCRIPT
Copy
// Install heapdump using npm: npm install --save-dev heapdump
const { createHeapDump } = require('heapdump');

setImmediate(() => {
  createHeapDump({ path: '/tmp/my.heap' })
    .then(path => console.log(`Created dump at ${path}`))
    .catch(err => console.error(err));
});
```

**Example input / output:**

```text
No specific input-output examples are needed here, as the main concern is debugging and fixing the cache implementation.
```

**Flow:**

1. Eviction Process
2. Cache Operations
3. Yes
4. No
5. Yes
6. Yes
7. No
8. Cache Operation
9. Is it a Get?
10. Update Access Order
11. Is Cache Full?
12. Evict Least Recently Used Entry
13. Set New Value
14. Is Eviction Needed?
15. Evict and Remove Reference
16. Set New Value

