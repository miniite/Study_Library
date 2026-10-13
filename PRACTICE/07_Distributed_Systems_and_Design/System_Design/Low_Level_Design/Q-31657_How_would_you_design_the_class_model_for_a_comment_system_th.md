---
type: "practice"
difficulty: "Deep"
---

# How would you design the class model for a comment system that must support threaded replies AND efficient 'load more' pagination at any depth without N+1 queries; where does the tree vs flat trade-off bite?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Threaded Comments Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Threaded_Comments_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust low-level class model for a comment system that supports both threaded replies and efficient "load more" pagination, we need to consider several key aspects: data structure, database schema, and query optimization. The goal is to ensure scalability while maintaining performance.

#### Triage

The question requires designing a class model for comments in a way that supports nested (threaded) replies efficiently and allows for efficient pagination at any depth without leading to N+1 queries. We need to explore trade-offs between using a tree structure versus a flat structure and identify potential pitfalls, especially with regard to query optimization.

Step-by-Step Solution
1. Data Model Overview

We will use a combination of a Comment entity and a nested set model for the threaded replies, combined with an efficient pagination strategy.

2. Entity Design

Let's define our primary entities:

PLAINTEXT
Comment {
    id: String,
    content: String,
    parent_id: String | null (null indicates top-level comments),
    author_id: String,
    created_at: Timestamp,
}

This basic structure allows for a tree-like hierarchy, where each comment can have zero or more children.

3. Nested Set Model

To support efficient querying of threaded replies, we use the nested set model. This involves adding additional fields to track the position of each node in the tree:

PLAINTEXT
Comment {
    id: String,
    content: String,
    parent_id: String | null (null indicates top-level comments),
    author_id: String,
    created_at: Timestamp,
    lft: Integer, // left boundary of the set
    rgt: Integer  // right boundary of the set
}

These fields help in quickly identifying a node’s subtree.

4. Query Optimization for Threaded Replies

To efficiently fetch replies at any depth, we need to leverage SQL techniques such as recursive Common Table Expressions (CTEs) or hierarchical queries:

SQL
WITH RECURSIVE thread AS (
    SELECT id, parent_id, content, author_id, created_at, lft, rgt
    FROM Comment
    WHERE id = 'root_comment_id'  -- replace with actual root comment ID

    UNION ALL

    SELECT c.id, c.parent_id, c.content, c.author_id, c.created_at, c.lft, c.rgt
    FROM Comment c
    JOIN thread t ON c.parent_id = t.id
)
SELECT * FROM thread;

This query will return all comments in the subtree of a given root comment.

5. Efficient Pagination Strategy

For efficient pagination, we can implement a combination of server-side and client-side techniques:

Server-Side: Use offset-based pagination with a limit to fetch the next set of comments.
Client-Side: Implement infinite scrolling or load more buttons that trigger additional requests to fetch new comments.

Here’s an example of how to fetch comments using offset-based pagination in SQL:

SQL
SELECT * FROM Comment WHERE id > 'last_fetched_comment_id' ORDER BY created_at DESC LIMIT 10;

This ensures that we avoid N+1 queries by fetching a batch of comments at once.

#### Trade-Offs
and Considerations
Tree Structure vs. Flat Structure
Tree Structure: Allows for natural hierarchical relationships but can lead to complex query logic, especially with recursive queries.
Flat Structure: Simplifies the database schema and querying process but loses the natural hierarchy, making it less intuitive for users.

#### Common Mistakes
and Follow-Ups
Ignoring Tree Queries: Forgetting to use efficient tree traversal techniques like CTEs can lead to performance degradation.
Not Handling Large Hierarchies Properly: Failing to consider edge cases where the comment tree is very deep or wide, which could impact query performance.

#### Closing Summary

By combining a nested set model with efficient pagination strategies and recursive queries, we can design a robust class model for comments that supports both threaded replies and efficient "load more" functionality without leading to N+1 queries. The trade-off between using a tree structure versus a flat one must be carefully considered based on the specific requirements of the application.

**Code:**

```

-- Example SQL for fetching a subtree with CTE WITH RECURSIVE thread AS ( SELECT id, parent_id, content, author_id, created_at, lft, rgt FROM Comment WHERE id = 'root_comment_id'

CODE
Copy
UNION ALL

SELECT c.id, c.parent_id, c.content, c.author_id, c.created_at, c.lft, c.rgt
FROM Comment c
JOIN thread t ON c.parent_id = t.id

) SELECT * FROM thread;

CODE
Copy

```sql
-- Example SQL for offset-based pagination
SELECT * FROM Comment WHERE id > 'last_fetched_comment_id' ORDER BY created_at DESC LIMIT 10;
```

**Flow:**

1. Yes
2. No
3. Fetch Root Comment
4. Is Child?
5. Fetch Children Recursively
6. Fetched All Comments
7. Fetch Subtree with CTE

