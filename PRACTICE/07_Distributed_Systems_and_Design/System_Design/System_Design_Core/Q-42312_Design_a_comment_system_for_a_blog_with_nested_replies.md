---
type: "practice"
difficulty: "Intermediate"
---

# Design a comment system for a blog with nested replies.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Chat History Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Chat_History_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR Summary

To design a comment system that supports nested replies, we need to consider a tree-like structure where each comment can have multiple children (replies). The key aspects are efficient storage and retrieval of comments, handling of recursive queries, and optimizing for both write and read operations.

#### Triaging the Question

The interviewer is probing your understanding of:

Data Structures: How you would represent hierarchical data.
Query Optimization: Efficient ways to retrieve nested comments.
Scalability and Performance: Handling a large number of comments and replies.
Database Design: Choosing an appropriate database schema for storage.

#### Assumptions
Each comment can have any number of children (replies).
Comments are version-controlled; each reply has a parent_id linking it to its parent comment.
The system should support pagination, especially in cases where there are many replies.

#### First Principles Solution

Database Schema

Comment
parent_id
Parent Comment
Children Comments

Data Model

We will use a relational database for this system, specifically storing comments and their relationships.

SQL
CREATE TABLE Comments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME ON UPDATE CURRENT_TIMESTAMP,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES Comments(id)
);

Query Optimization

To retrieve a comment and all its replies, we can use a recursive query or a series of queries.

SQL
SELECT * FROM Comments WHERE id = :commentId;
SELECT * FROM Comments WHERE parent_id = :commentId ORDER BY created_at DESC LIMIT 10; -- pagination limit

Pagination and Load Balancing

For large discussions, implement pagination to fetch a limited number of comments at a time.

API Design

Define RESTful endpoints for creating, reading, updating, and deleting comments.

#### Closing Summary

In summary, designing a nested comment system involves setting up an appropriate database schema, optimizing queries, handling recursion, implementing pagination, and ensuring efficient API design. This solution ensures both scalability and performance, making it robust for blogs with extensive discussion threads.

#### Common Mistakes
and Follow-Ups

Mistake: Overlooking the need for recursive querying in a nested structure.

#### Follow-Up
Explain how to implement recursive queries using SQL features like WITH RECURSIVE.

Mistake: Ignoring pagination without considering performance implications on large datasets.

#### Follow-Up
Discuss the trade-offs between fetching all comments at once versus loading them lazily.

Mistake: Not considering concurrency and locking issues when multiple users interact with the same comment thread.

#### Follow-Up
Explain how to use database transactions and locks to handle concurrent writes safely.

**Comparison:**

| Comments |
| ---      |
| id       | INT   | PRIMARY KEY, AUTO_INCREMENT         |
| content  | TEXT  | NOT NULL                            |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP    |
| updated_at | DATETIME | ON UPDATE CURRENT_TIMESTAMP        |
| parent_id | INT    | FOREIGN KEY (parent_id) REFERENCES Comments(id) |

**Code:**

```sql
-- Example SQL for creating a comments table
CREATE TABLE Comments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME ON UPDATE CURRENT_TIMESTAMP,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES Comments(id)
);
```

**Example input / output:**

```text
Input:
Retrieve all replies to comment with id = 1, paginated by 5.

Output:
[
  {
    "id": 2,
    "parent_id": 1,
    "content": "Thank you! I'm glad you found it helpful.",
    "created_at": "2023-10-06",
    "updated_at": "2023-10-06"
  },
  {
    "id": 4,
    "parent_id": 2,
    "content": "Sure, it's available on GitHub.",
    "created_at": "2023-10-08",
    "updated_at": "2023-10-08"
  }
]
```

**Data:**

```json
{
  "Comments": [
    {
      "id": 1,
      "content": "Great article!",
      "created_at": "2023-10-05",
      "updated_at": "2023-10-05"
    },
    {
      "id": 2,
      "parent_id": 1,
      "content": "Thank you! I'm glad you found it helpful.",
      "created_at": "2023-10-06",
      "updated_at": "2023-10-06"
    },
    {
      "id": 3,
      "parent_id": 1,
      "content": "Can I get the source code?",
      "created_at": "2023-10-07",
      "updated_at": "2023-10-07"
    },
    {
      "id": 4,
      "parent_id": 2,
      "content": "Sure, it's available on GitHub.",
      "created_at": "2023-10-08",
      "updated_at": "2023-10-08"
    }
  ]
}
```

**Flow:**

1. User creates a comment
2. Insert into Comments table
3. Retrieve comments and repliesfor a specific ID
4. Fetch top-level comment
5. Recursive query to fetch allchildren (replies)
6. Return results with pagination
7. Handle concurrency usingtransactions
8. Publish updates/notificationsas needed

