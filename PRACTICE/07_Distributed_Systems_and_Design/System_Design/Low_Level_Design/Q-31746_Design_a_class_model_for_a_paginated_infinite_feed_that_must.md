---
type: "practice"
difficulty: "Deep"
---

# Design a class model for a paginated infinite feed that must stay stable (no dupes/skips) even as new items are inserted at the top while a user scrolls; how does cursor design solve what offset can't?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Consumer Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In designing a class model for a paginated infinite feed, we need to ensure stability and prevent duplicates or skips in the feed as new items are inserted at the top while users scroll. A robust solution involves using cursors instead of offsets. This approach allows us to maintain consistency by leveraging unique identifiers that track the position within the data stream.

#### Triaging the Question

The core of this question revolves around designing a system where items can be efficiently paginated and displayed in real-time without duplicates or skips, even as new items are added dynamically at the top. The interviewer is probing your understanding of low-level design principles, specifically how to handle dynamic updates while maintaining data integrity.

#### First-Principles Solution

To achieve this, we will employ a cursor-based approach instead of using offsets for pagination. Here’s why:

Stability: Offsets can become invalid when new items are inserted at the top because they rely on static positions which change with each insertion.
Scalability and Performance: Cursors allow us to track the position within the data stream more efficiently, especially in large datasets.
Step-by-Step Solution

Define the Data Model

Each item in the feed will have a unique identifier (ID).
We’ll use a cursor that references the next valid item ID or a specific index within the list.

Cursor-Based Pagination Logic

When a user scrolls to the bottom, we fetch more items starting from the last seen cursor.
As new items are inserted at the top, they will be appended with their own unique IDs and updated cursors.

#### Implementation
Details

Store the current cursor in the client state or session.
On each fetch request, pass this cursor to the server along with a limit parameter.
The server processes the request by returning items starting from the provided cursor, ensuring no duplicates are returned.
Code Example
PYTHON
class FeedItem:
    def __init__(self, id, content):
        self.id = id
        self.content = content

class InfiniteFeedService:
    def __init__(self):
        self.items = []  # Simulating a large dataset of items with unique IDs

    def add_item(self, item: FeedItem):
        """Add an item to the feed and update cursors accordingly."""
        # Append new item at the top
        self.items.insert(0, item)

        # Update cursors for existing items (if any)
        for i in range(len(self.items) - 1, 0, -1):
            self.items[i].cursor = self.items[i-1].id

    def get_items_from_cursor(self, cursor: str, limit: int):
        """Fetch items from the feed starting from the provided cursor."""
        items_to_return = []

        for item in self.items:
            if item.id == cursor:
                break
            items_to_return.append(item)

        # Ensure we don't return more than 'limit' items and avoid duplicates
        return [item.content for item in items_to_return[-limit:]]

#### Closing Summary

By using cursors, we can ensure that our infinite feed remains stable even as new items are inserted at the top. This approach provides a more robust solution compared to traditional offset-based pagination. It ensures consistency and prevents duplicates or skips by dynamically tracking positions within the data stream.

#### Common Mistakes
and Follow-Ups
Ignoring Duplicates: Ensure you handle duplicate IDs properly, especially if new items have the same ID as existing ones.

#### Performance Considerations
In very large feeds, consider caching strategies to reduce fetch times.
Edge Cases: Handle scenarios where no cursor is provided or invalid cursors are passed.

Would you like to explore further aspects of this design, such as integration with a database or additional optimizations?

**Comparison:**

```
Item ID	Content
1	Item 1
2	Item 2
3	Item 3
New Item Added at Top
{"id": "0", "content": "New Item"}
Updated Feed after Addition
{"id": "0", "content": "New Item"}
{"id": "1", "content": "Item 1"}
{"id": "2", "content": "Item 2"}
{"id": "3", "content": "Item 3"}
Cursor Values After Addition
id: "0" -> cursor: "1"
id: "1" -> cursor: "2"
id: "2" -> cursor: "3"
```

**Code:**

```python
class FeedItem:
    def __init__(self, id, content):
        self.id = id
        self.content = content

class InfiniteFeedService:
    def __init__(self):
        self.items = []  # Simulating a large dataset of items with unique IDs

    def add_item(self, item: FeedItem):
        """Add an item to the feed and update cursors accordingly."""
        # Append new item at the top
        self.items.insert(0, item)

        # Update cursors for existing items (if any)
        for i in range(len(self.items) - 1, 0, -1):
            self.items[i].cursor = self.items[i-1].id

    def get_items_from_cursor(self, cursor: str, limit: int):
        """Fetch items from the feed starting from the provided cursor."""
        items_to_return = []

        for item in self.items:
            if item.id == cursor:
                break
            items_to_return.append(item)

        # Ensure we don't return more than 'limit' items and avoid duplicates
        return [item.content for item in items_to_return[-limit:]]
```

**Example input / output:**

```text
Input:
- Cursor: "2"
- Limit: 2

Output:
- ["Item 1"]
```

**Data:**

```json
{
  "items": [
    {"id": "1", "content": "Item 1"},
    {"id": "2", "content": "Item 2"},
    {"id": "3", "content": "Item 3"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Scrolls to Bottom
4. Is Cursor Valid?
5. Fetch Items from Cursor
6. Return Items
7. Fetch Latest Data and UpdateCursor
8. Add New Item at Top
9. Update Cursors for ExistingItems
10. Set New Item's Cursor toPrevious Item's ID
11. Ensure No Duplicates or Skips

