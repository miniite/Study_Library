---
type: "practice"
difficulty: "Deep"
---

# Design an infinite-scroll feed component. What state does it track and how do you avoid re-fetch loops?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent Consumer Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Infinite-scroll feed components are a common pattern in web applications where content is fetched progressively from a server as a user scrolls down a page. The key challenge here is to design the state tracking and data fetching mechanism to ensure smooth performance and prevent re-fetch loops, which can cause data duplication or wasted requests.

#### Triaging the Question

The interviewer is likely probing your understanding of state management in React components, efficient data fetching strategies, and how to implement a seamless user experience with infinite scrolling. The question also touches on potential pitfalls such as re-fetching data that has already been loaded, leading to performance issues and unnecessary server load.

#### First-Principles Solution

To design an effective infinite-scroll feed component, we need to track the state of which items have been fetched so far and ensure that subsequent fetches only retrieve new content. Here are the key steps:

State Management: Track the current page number or offset in the data.
Fetching Strategy: Use server-side pagination to fetch a fixed number of records per request.
Throttling: Ensure smooth user interaction by throttling scroll events and batching requests.
Error Handling: Handle network errors gracefully without interrupting the scrolling experience.
Step-by-Step Solution
Component State:
We need to keep track of the current page number, the total number of items fetched so far, and whether there is more data to load.
Fetching Logic:
When the user reaches the bottom of the scrollable content, we trigger a request to fetch new data from the server.
Throttling Scroll Events:
To prevent multiple fetch requests while the user is still scrolling, we can throttle the scroll event using techniques like lodash.debounce.
Server-Side Pagination:
The server should respond with the requested number of items starting at the last known offset.
Error Handling and Re-fetch Prevention:
If a fetch request fails, retry after a reasonable timeout.
Ensure that only new data is added to the existing list to prevent duplication.
Detailed Implementation
JAVASCRIPT
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import _ from 'lodash';

#### cons
t InfiniteScrollFeed = () => {

#### cons
t [items, setItems] = useState([]);

#### cons
t [fetching, setFetching] = useState(false);

#### cons
t [page, setPage] = useState(1);

  // Throttle the scroll event

#### cons
t handleScroll = _.debounce(() => {
    if (window.innerHeight + document.documentElement.scrollTop !== document.documentElement.offsetHeight) return;

    // Fetch more data only if there's more to load
    if (!fetching && items.length > 0 && items[items.length - 1].id < 100) {
      setFetching(true);

      axios.get(`/api/items?page=${page}`)
        .then(response => {

#### cons
t newItems = response.data;
          setItems([...items, ...newItems]);
          setPage(page + 1);
          setFetching(false);
        })
        .catch(error => {

#### cons
ole.error('Error fetching data:', error);
          // Retry after a delay if needed
          setTimeout(() => fetchMoreData(), 5000);
        });
    }
  }, 200);

  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

#### cons
t renderItems = items.map(item => (
    <div key={item.id}>{item.name}</div>
  ));

  return (
    <div>
      {renderItems}
    </div>
  );
};

export default InfiniteScrollFeed;

#### Closing Summary
and Common Mistakes/Follow-Ups

Infinite-scroll feed components require careful state management to ensure a smooth user experience. The key is to track the current page number, handle server-side pagination efficiently, and avoid re-fetch loops by only adding new data to the existing list. Common mistakes include failing to throttle scroll events or not handling network errors gracefully, leading to unnecessary requests or poor performance.

#### Follow-up
questions might explore how you would handle different screen sizes, implement infinite loading for images, or optimize the component for large datasets.

**Code:**

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import _ from 'lodash';

const InfiniteScrollFeed = () => {
  const [items, setItems] = useState([]);
  const [fetching, setFetching] = useState(false);
  const [page, setPage] = useState(1);

  // Throttle the scroll event
  const handleScroll = _.debounce(() => {
    if (window.innerHeight + document.documentElement.scrollTop !== document.documentElement.offsetHeight) return;

    // Fetch more data only if there's more to load
    if (!fetching && items.length > 0 && items[items.length - 1].id < 100) {
      setFetching(true);

      axios.get(`/api/items?page=${page}`)
        .then(response => {
          const newItems = response.data;
          setItems([...items, ...newItems]);
          setPage(page + 1);
          setFetching(false);
        })
        .catch(error => {
          console.error('Error fetching data:', error);
          // Retry after a delay if needed
          setTimeout(() => fetchMoreData(), 5000);
        });
    }
  }, 200);

  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  const renderItems = items.map(item => (
    <div key={item.id}>{item.name}</div>
  ));

  return (
    <div>
      {renderItems}
    </div>
  );
};

export default InfiniteScrollFeed;
```

**Example input / output:**

```text
Input:
- User scrolls to the bottom of a page.
Output:
- New data is fetched from the server and appended to the existing list without reloading or duplicating existing content.
```

**Data:**

```json
{
  "items": [
    {"id": 1, "name": "Item A"},
    {"id": 2, "name": "Item B"},
    // ... more items
  ]
}
```

