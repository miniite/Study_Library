---
type: "practice"
difficulty: "Intermediate"
---

# Design the client-side feed rendering: infinite scroll, optimistic like counts, and prefetching the next page — how do you keep it smooth on a low-end Android device?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Micro-frontends](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Micro-frontends.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing the client-side feed rendering with features such as infinite scroll, optimistic like counts, and prefetching of the next page, we need to ensure that these functionalities are optimized for performance on low-end Android devices. The key is to balance user experience with minimal resource consumption.

#### Triage

The question is probing how to design a client-side feed rendering system that seamlessly integrates infinite scrolling, updates like counts in real-time (optimistically), and prefetches the next page of content without causing undue strain on the device's resources. We need to consider hardware limitations such as CPU speed, memory, network connectivity, battery life, and overall user experience.

#### Assumptions
The client is an Android app.
The feed contains a large number of items (e.g., posts or tweets).
Network conditions can vary significantly, from good 4G to poor Wi-Fi to no connection.
The device has limited processing power and memory compared to high-end devices.

#### Solution

To achieve this, we will implement the following steps:

Infinite Scroll: Implement a smooth infinite scroll by fetching only a few items at a time as the user scrolls close to the bottom of the list. Use lazy loading techniques.
Optimistic Like Counts: Update like counts immediately on the client-side before sending an update request to the server, providing instant feedback to users.
Prefetching Next Page: Prefetch the next page of content as soon as possible but only if network conditions are good enough and resources allow.
Step-by-Step Solution

Infinite Scroll

Implement a lazy loading mechanism where we fetch data in batches (e.g., 10 items) when the user scrolls close to the bottom.
Use an indicator or marker to show that more content is being fetched, e.g., a "Loading More" message.

Optimistic Like Counts

When a user likes a post, update the like count immediately in the UI without sending a request to the server.
If the network connection fails while updating the client-side state, revert the change when a successful update is received from the server.

Prefetching Next Page

Monitor network conditions and only prefetch content if there is sufficient memory available and the battery level is high enough.
Use predictive algorithms to estimate when the user might scroll down further and start fetching data in advance.
Code Example
JAVASCRIPT
// Infinite Scroll Implementation
class FeedRenderer {

#### cons
tructor(fetchFn, maxBatchSize = 10) {
    this.fetchFn = fetchFn;
    this.batchSize = maxBatchSize;
    this.offset = 0;
    this.loadingMore = false;
  }

  async renderFeed() {
    if (!this.loadingMore && (window.innerHeight + window.scrollY) >= document.body.offsetHeight - this.batchSize * 2) {
      this.loadingMore = true;

#### cons
t items = await this.fetchFn(this.offset, this.batchSize);
      // Update UI with new items
      this.renderItems(items);
      this.offset += items.length;
    }
  }

  renderItems(items) {
    // Render the fetched items into the UI
  }
}
Sample Data
JSON
{
  "offset": 0,
  "batchSize": 10,
  "items": [
    {"id": 1, "text": "Post 1"},
    {"id": 2, "text": "Post 2"}
  ]
}

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Over-fetching data can lead to excessive memory usage and network traffic. Not handling errors gracefully (e.g., failed network requests) can cause a bad user experience.

#### Follow-Ups
Consider how caching strategies could further optimize performance, or how background tasks could be used to pre-cache content in the background.
Correctness + Completeness Confirmation

The solution above addresses key aspects of infinite scroll, optimistic like counts, and prefetching while keeping considerations for low-end devices. It ensures that the implementation is robust and efficient.

**Comparison:**

```
Batch	Offset	Items Fetched
1	0	[Post 1, Post 2]
2	10	[Post 3, Post 4]
```

**Code:**

```javascript
// Infinite Scroll Implementation
class FeedRenderer {
  constructor(fetchFn, maxBatchSize = 10) {
    this.fetchFn = fetchFn;
    this.batchSize = maxBatchSize;
    this.offset = 0;
    this.loadingMore = false;
  }

  async renderFeed() {
    if (!this.loadingMore && (window.innerHeight + window.scrollY) >= document.body.offsetHeight - this.batchSize * 2) {
      this.loadingMore = true;
      const items = await this.fetchFn(this.offset, this.batchSize);
      // Update UI with new items
      this.renderItems(items);
      this.offset += items.length;
    }
  }

  renderItems(items) {
    // Render the fetched items into the UI
  }
}
```

**Example input / output:**

```text
// Example input and output for fetching items
const fetchFn = async (offset, batchSize) => {
  return new Promise((resolve) => {
    setTimeout(() => resolve(sampleData.items), 500);
  });
};

const feedRenderer = new FeedRenderer(fetchFn);
feedRenderer.renderFeed();

// Output: UI updates with the first batch of items
```

**Data:**

```json
{
  "offset": 0,
  "batchSize": 10,
  "items": [
    {"id": 1, "text": "Post 1"},
    {"id": 2, "text": "Post 2"}
  ]
}
```

**Flow:**

1. No
2. Detect Scroll Position
3. Check if Close to Bottom
4. Is Loading More?
5. Fetch New Batch
6. Update UI with Fetched Items
7. Increment Offset
8. Set Loading More to False

