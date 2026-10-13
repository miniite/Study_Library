---
type: "practice"
difficulty: "Deep"
---

# You're deciding the paved-road default for pagination across every list API in the company. Keyset is faster but breaks 'jump to page N.' How do you set an org-wide standard that balances performance, product needs, and migration cost of legacy OFFSET endpoints?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Offset Pagination Pitfalls](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Offset_Pagination_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding on a default pagination method for list APIs across your organization, you must balance several factors including performance, user experience, compatibility with existing endpoints, and the effort required to migrate from one method to another. The key decision involves choosing between using a keyset pagination approach (which is faster but breaks 'jump to page N') or sticking with the traditional OFFSET pagination, which supports jumping to any page but can be less performant.

To set an org-wide standard, we need to analyze these factors and make informed trade-offs. Let's break down the problem from first principles:

Understanding Keyset vs OFFSET Pagination:

Keyset Pagination: Uses a unique identifier or timestamp from one of the records returned in the previous page request as a starting point for the next page.
OFFSET Pagination: Skips a fixed number of rows (defined by offset) and returns the next set of rows.

Impact on Performance:

Keyset pagination typically outperforms OFFSET pagination because it doesn't require scanning through all previous records to find the offset.
However, if there are frequent jumps between pages, keyset pagination can introduce additional complexity in maintaining the correct starting point for each request.

User Experience Considerations:

Users expect to be able to jump directly to any page without having to navigate sequentially through all previous records.
Keyset pagination might lead to a more seamless user experience if users are frequently jumping between pages, but it can be cumbersome when the sequence is needed.

Compatibility and Migration Costs:

OFFSET pagination has been the standard in many legacy systems, making migration costly due to potential changes required throughout the codebase.
Keyset pagination might introduce new complexity for developers, especially if they are not familiar with the technique.

Given these considerations, we can set an org-wide standard that balances performance and user experience while managing the cost of migration. Here’s a step-by-step approach:

Default to Keyset Pagination:

Implement keyset pagination as the default because it offers better performance and scalability.
Encourage developers to use this method in new projects.

Maintain Compatibility with OFFSET:

For existing OFFSET-based APIs, ensure that a backward compatibility layer is provided to allow seamless migration over time.
Provide clear documentation on how to transition from OFFSET to keyset pagination.

Migration Plan for Legacy Systems:

Develop a phased migration plan that prioritizes critical systems first.
Offer training and support resources to help teams understand and implement the new pagination method.

Performance Monitoring and Optimization:

Monitor the performance of list APIs using both methods in production environments.
Use performance metrics to identify areas where keyset pagination can be further optimized or where legacy OFFSET pagination might still serve well.

User Experience Improvements:

Develop tools or middleware that can dynamically switch between different pagination strategies based on user actions, optimizing for both performance and user convenience.
Continuously gather feedback from users to ensure the chosen strategy meets their needs.

By adopting a balanced approach, you can ensure that your organization's APIs are optimized for performance while maintaining backward compatibility. This standard will help in reducing technical debt and ensuring a smooth transition over time.

**Code:**

```python
# Example of keyset pagination implementation in Python

def fetch_records_after_keyset(offset_id):
    # Query database using offset_id to get next set of records
    pass

def update_pagination_context(records):
    # Extract the last record's id or timestamp for use as the new offset_id
    return records[-1].id
```

**Example input / output:**

```text
Input:
- API request for page 5 with keyset pagination, using lastRecordId from previous response.

Output:
- List of records starting after the record identified by 'lastRecordId'.
- Updated pagination context including new lastRecordId.
```

**Data:**

```json
{
  "offsetPagination": {
    "currentOffset": 0,
    "pageSize": 10
  },
  "keysetPagination": {
    "lastRecordId": "abc123"
  }
}
```

