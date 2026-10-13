---
type: "practice"
difficulty: "Deep"
---

# How do you reason about the complexity of choosing keyset (cursor) pagination over offset for a deep-scroll feed API, including how sort keys and deletes affect the O(log n) seek guarantee?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exact vs Approximate Counting](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Exact_vs_Approximate_Counting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a backend service for a deep-scroll feed API, one of the critical decisions is choosing between keyset (cursor) pagination and offset-based pagination. Keyset pagination offers several advantages over offset-based pagination, particularly in terms of time and space complexity management. In this answer, we'll explore how to reason about these complexities, especially considering sort keys and deletions.

#### Triage
Understanding the Question

The question is essentially asking us to compare the time and space complexity of using a keyset (cursor) pagination approach versus an offset-based pagination for deep-scroll feed APIs. The focus is on understanding how sorting mechanisms and deletions affect the efficiency and guarantees provided by the keyset method, specifically its O(log n) seek guarantee.

First-Principles Reasoning
Keyset Pagination vs. Offset-Based Pagination

Offset-Based Pagination:

Time Complexity: Typically O(n/k), where n is the total number of items and k is the page size.
Space Complexity: Minimal, as it only stores a small integer offset.

Keyset (Cursor) Pagination:

Time Complexity: Generally O(log n + k), where n is the total number of items, and k is the page size. This complexity comes from both the seek operation to find the key and then reading the subsequent k items.
Space Complexity: Higher, as it requires storing keys or cursors for each item.
Impact of Sort Keys

In a deep-scroll feed API, items are often sorted by time or other criteria. Keyset pagination leverages this sorting to efficiently locate the next page. The O(log n) seek guarantee means that the system can quickly find the correct cursor position, ensuring efficient pagination performance.

Deletions and Their Impact

Deletions in Offset-Based Pagination:

When items are deleted, the offset-based approach remains robust because it doesn't rely on item keys.
However, it may require additional logic to skip over deleted entries, potentially degrading performance slightly.

Deletions in Keyset Pagination:

Deleting an item can invalidate existing cursors if those cursors point directly at the deleted item or its vicinity.
To handle deletions, systems often implement mechanisms like tombstones (markers for deleted items) or use additional metadata to reindex keys.
Detailed Analysis
Time Complexity Analysis
Seek Operation: The keyset approach uses a seek operation, which can take O(log n) time if the data structure is balanced. For instance, in an indexed database, this might involve B-tree operations.
Read Operation: Once the correct cursor position is found, reading k items takes O(k).
Space Complexity Analysis
Offset-Based Pagination:
Minimal overhead, storing only a small integer value per page request.
Keyset Pagination:
Higher overhead due to storage of keys or cursors for each item. This can be managed by using efficient data structures like B-trees or hash maps.
Practical Considerations
Scalability: Keyset pagination scales better with large datasets because the seek operation is logarithmic, whereas offset-based pagination becomes less efficient as n increases.
User Experience: Users prefer smooth scrolling and immediate loading of content without gaps or delays, which keyset pagination often provides.

#### Closing Summary

Keyset pagination offers a more scalable solution for deep-scroll feed APIs, with O(log n) seek guarantees, making it robust against deletions and providing efficient performance at the cost of higher storage requirements. While offset-based pagination is simpler and has lower space complexity, keyset pagination's efficiency in handling large datasets makes it a better choice for complex systems.

#### Common Mistakes
and Follow-Ups
Overlooking Deletions: Failing to account for the impact of deletions on existing cursors can lead to incorrect or inefficient pagination.
Ignoring Seek Overhead: Assuming that seek operations are constant time without understanding their actual complexity (e.g., B-tree operations).
Neglecting User Experience: Not considering how page loading times and smooth scrolling affect user satisfaction.

**Comparison:**

```
Item ID	Content
001	First Item
002	Second Item
...	...
999	Last Item
```

**Code:**

```python
# Example code for keyset pagination in Python using a simple dictionary as the data store

class CursorPagination:
    def __init__(self, items):
        self.items = items

    def get_page(self, cursor=None, page_size=10):
        if not cursor:
            return (None, [])

        start_index = 0
        for i, item in enumerate(self.items):
            if item['id'] == cursor:
                break
        else:  # If we didn't find a match, return the last page
            start_index = len(self.items) - page_size

        end_index = min(start_index + page_size, len(self.items))

        next_cursor = self.items[end_index-1]['id'] if end_index > start_index else None
        return (next_cursor, self.items[start_index:end_index])
```

**Example input / output:**

```text
- **Input**: `get_page(cursor="050", page_size=10)`
- **Output**: `(next_cursor="147", items=[{"id": "051", "content": "Item 51"}, {"id": "052", "content": "Item 52"}, ...])`
```

**Data:**

```json
[
  {"id": "001", "content": "First Item"},
  {"id": "002", "content": "Second Item"},
  ...
  {"id": "999", "content": "Last Item"}
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is Cursor Provided?
7. Find Start Index Using Cursor
8. Calculate End Index
9. Retrieve Items from Start toEnd Index
10. Generate Next Cursor ifPossible
11. Is There More Data To Fetch?
12. End
13. Return Last Page

