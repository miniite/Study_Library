---
type: "practice"
difficulty: "Deep"
---

# A search-as-you-type endpoint gets hammered and its trie rebuild blocks the event loop; how do you move the build off the hot path and serve reads from an immutable snapshot?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Tries and Prefix Matching](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Tries_and_Prefix_Matching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a search-as-you-type endpoint being overwhelmed by frequent trie builds, we need to decouple the write operations (trie rebuilds) from the read operations (searches). This can be achieved by using immutable snapshots and leveraging background processing. Here’s how you can approach this problem:

TL;DR

The key solution involves splitting the data into immutable snapshots, which are updated asynchronously in a background process. Reads can then safely serve queries from these snapshots without blocking the event loop.

#### Triaging the Question
Understanding the Problem: The current implementation of the search-as-you-type endpoint blocks the event loop when rebuilding the trie due to frequent updates.
What is Being Probed:
Knowledge of data structures and their trade-offs (trie vs others).
Understanding of concurrency and asynchronous programming.
Experience with immutable data structures and their benefits.

#### Assumptions
The system has a limited number of concurrent updates to the trie, allowing for background processing.
We need a solution that minimizes downtime during high traffic periods.

#### First-Principles Solution
Immutable Snapshots: Create immutable copies (snapshots) of the trie data structure every time there is an update. This ensures that the read operations do not block while writes are happening.
Background Processing: Use a background worker to handle trie rebuilds, ensuring that these do not affect the main event loop where read requests are served.
Step-by-Step Implementation

Immutable Trie Data Structure:

Each update (insertion or deletion) will create a new trie node with references to the previous state.
This allows multiple snapshots of the same trie, each representing different states at various points in time.

Background Processing:

Implement a background worker that handles updates and creates new trie snapshots.
The main application thread remains unblocked, allowing it to serve read requests efficiently from any snapshot.

Serving Reads from Snapshots:

Each search-as-you-type request is served by the latest available immutable snapshot.
If a new update occurs between two requests, the response might be slightly outdated but will not block other requests.
Detailed Example

Let’s assume you have an ImmutableTrie class that supports creating snapshots and performing searches. Here’s a simplified example:

PYTHON
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word_end = False

class ImmutableTrie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_word_end = True

    def search(self, prefix):
        # Search the trie and return all words with given prefix
        pass

# Background Worker
import asyncio

async def background_worker(trie):
    while True:
        word_to_insert = await get_next_update()  # Assume this fetches next update async
        new_snapshot = trie.clone_with_update(word_to_insert)
        trie.update(new_snapshot)

# Main Application Loop
from concurrent.futures import ThreadPoolExecutor

def main():
    trie = ImmutableTrie()
    background_executor = ThreadPoolExecutor(max_workers=1)
    asyncio.get_event_loop().run_in_executor(background_executor, background_worker, trie)

if __name__ == "__main__":
    main()

#### Closing Summary
and Common Mistakes
Immutable Snapshots: Ensure that updates are handled asynchronously to avoid blocking the event loop.
Background Processing: Use a separate thread or process to handle trie rebuilds, allowing main application threads to serve reads efficiently.

#### Cons
istency: Be mindful of consistency between updates and queries. Outdated data might be acceptable for high-frequency searches but should not affect critical operations.

#### Common Mistakes
Blocking Main Thread: Failing to use background processing can lead to the main thread being blocked by trie rebuilds.
Insufficient Testing: Not thoroughly testing different update frequencies and their impact on performance.

**Code:**

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word_end = False

class ImmutableTrie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_word_end = True

# Simplified background worker and main loop example
import asyncio

async def background_worker(trie):
    while True:
        word_to_insert = await get_next_update()  # Assume this fetches next update async
        new_snapshot = trie.clone_with_update(word_to_insert)
        trie.update(new_snapshot)

def main():
    trie = ImmutableTrie()
    background_executor = ThreadPoolExecutor(max_workers=1)
    asyncio.get_event_loop().run_in_executor(background_executor, background_worker, trie)

if __name__ == "__main__":
    main()
```

**Flow:**

1. User Inserts
2. Background Worker UpdatesTrie
3. User Search Request
4. Serve from Latest ImmutableSnapshot
5. Background Update Complete
6. New Snapshot Created andServe Reads

