---
type: "practice"
difficulty: "Deep"
---

# A REST endpoint occasionally returns duplicate items in a paginated list under concurrent writes because it sorts by a mutable field. How do you diagnose this and design a stable sort key that survives updates?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Consistent Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose and fix the issue of duplicate items appearing in a paginated list due to concurrent writes, we need to understand how sorting by mutable fields can lead to instability. The root cause is likely related to the concurrent modification of these mutable fields during data updates. Our goal is to design a stable sort key that remains consistent even when multiple concurrent operations are modifying the underlying data.

#### Triage
and Assumptions
Problem Understanding: The endpoint returns paginated lists, but occasionally shows duplicate items because it sorts by a mutable field.

#### Assumptions
The sorting is done on one or more fields that can change (mutable).
Concurrency issues arise from multiple concurrent writes modifying the same data.
We are dealing with a stateful system where each item has some mutable field used for sorting.
Step-by-Step Solution
Identify the Issue
The primary issue is that the mutable fields being used for sorting can change during the time it takes to fetch and return the paginated data, leading to inconsistencies.
Stable Sort Key Design
To address this, we need to ensure that the sort key remains stable even if the underlying field values are updated.

#### Approach
Combining Unique Identifiers with Timestamps

Unique Identifier + Timestamp:

Use a combination of a unique identifier and a timestamp as the primary sort key.

#### Implementation
Steps:

PYTHON
class Item:
    def __init__(self, id, mutable_field, timestamp):
        self.id = id
        self.mutable_field = mutable_field
        self.timestamp = timestamp

# Example item creation
items = [
    Item(1, 10, "2023-10-01T12:00:00Z"),
    Item(2, 5, "2023-10-01T12:00:00Z"),
    Item(3, 7, "2023-10-01T12:00:00Z")
]

Sorting Logic:

During retrieval and sorting, use a composite key that includes both the unique identifier and the timestamp.
PYTHON
def sort_items_by_stable_key(items):
    return sorted(items, key=lambda x: (x.id, x.timestamp))

Concurrent Writes Handling:

Ensure that when an item is updated, its timestamp also gets updated to reflect the new version.
PYTHON
# Example update logic
def update_item(item_id, new_value):
    for item in items:
        if item.id == item_id:
            item.timestamp = "2023-10-02T12:00:00Z"  # Updated timestamp
            item.mutable_field = new_value
            break

API Implementation:

Modify the API to include both the unique identifier and the timestamp in the sort key.
PYTHON
def get_paginated_items(page_size, page_number):
    start_index = (page_number - 1) * page_size
    end_index = start_index + page_size

    sorted_items = sort_items_by_stable_key(items)
    paginated_items = sorted_items[start_index:end_index]
    return [item.id for item in paginated_items]  # Return only the IDs for simplicity

#### Closing Summary
and Common Mistakes
Summary: By combining a unique identifier with a timestamp, we can ensure that even if mutable fields are updated concurrently, the sort order remains stable.

#### Common Mistakes
Failing to account for concurrent updates: Ensure that your system handles timestamps correctly during updates.
Ignoring uniqueness: Make sure each item has a unique ID to avoid ambiguity in sorting.
Not considering edge cases: Test scenarios where multiple items have the same mutable field and timestamp values.
Additional Considerations
Performance: Combining fields for sorting can increase query complexity. Ensure that performance is not significantly impacted.
Database Constraints: Implement database constraints or indexes on unique identifiers and timestamps to optimize read operations.

