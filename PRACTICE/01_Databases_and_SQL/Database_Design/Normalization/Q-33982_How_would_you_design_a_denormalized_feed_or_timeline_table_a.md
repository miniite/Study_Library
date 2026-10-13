---
type: "practice"
difficulty: "Expert"
---

# How would you design a denormalized feed or timeline table and manage fan-out on write versus fan-out on read?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Denormalization and Locking](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/Denormalization_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a denormalized feed or timeline table, we need to balance the trade-offs between read performance and data consistency. As a Database Developer Architect/Principal, I'll break down this problem from first principles, providing a step-by-step solution that covers key concepts like fan-out on write versus fan-out on read.

The core challenge is managing writes (fan-out on write) while ensuring efficient reads (fan-out on read). Fan-out on write means distributing data to multiple tables/views upon insert or update, whereas fan-out on read involves fetching data from multiple sources when a user requests it. Both approaches have their merits and drawbacks, which we will explore in detail.

First, let's define the problem:

TL;DR: We need to design a denormalized timeline table that efficiently handles both write operations (fan-out) and read operations (fan-out). The goal is to balance performance with consistency.

#### Triaging the Question

The question requires understanding how to manage fan-out on write versus fan-out on read. Fan-out on write can be complex due to the need for distributed transactions or compensating actions, while fan-out on read can cause contention and increased complexity in query logic. We'll need to carefully balance these aspects.

#### Assumptions
The system handles a high volume of writes (e.g., social media updates).
Reads are more frequent than writes.
Data consistency is critical for writes but less so for reads.
The system supports distributed computing and can use external services if necessary.

#### Solution
from First Principles
1. Define the Denormalized Table Schema

The timeline table should include key fields like user ID, post ID, timestamp, content, etc., plus additional fields to denormalize data such as follower counts or profile information for quick access.

SQL
CREATE TABLE Timeline (
    PostID INT PRIMARY KEY,
    UserID INT NOT NULL,
    Timestamp TIMESTAMP NOT NULL,
    Content TEXT NOT NULL,
    FollowerCount INT DEFAULT 0,
    ProfileName VARCHAR(100)
);
2. Fan-Out on Write

Fan-out on write involves distributing data to multiple tables or views when a user creates or updates content.

#### Approach
Create materialized views or copy rows to other relevant tables/views. For example, if a user follows another user, update the follower count in the Timeline table and notify related followers' timeline feeds.

SQL
-- Example of fan-out on write for a follow action
INSERT INTO Timeline (PostID, UserID, Timestamp, Content, FollowerCount, ProfileName)
VALUES (:new_post_id, :user_id, CURRENT_TIMESTAMP(), 'New Post', 0, :profile_name);

UPDATE Followers
SET FollowedUserCount = FollowedUserCount + 1
WHERE FollowerUserID = :user_id;

-- Notify followers' timelines if necessary
3. Fan-Out on Read

Fan-out on read involves fetching data from multiple sources when a user views their timeline or another user's posts.

#### Approach
Use caching, indexing, and efficient queries to minimize latency. Consider using distributed cache services like Redis for real-time updates and query optimization techniques.

SQL
SELECT PostID, UserID, Timestamp, Content, FollowerCount, ProfileName
FROM Timeline
WHERE UserID = :user_id OR FollowedUserID = :user_id;

-- Example of fetching followers' timelines
SELECT T.PostID, T.UserID, T.Timestamp, T.Content, T.FollowerCount, T.ProfileName
FROM Timeline AS T
JOIN Followers ON T.UserID = FollowedUserID AND FollowerUserID = :user_id;
4. Balancing Fan-Outs and Consistency

To balance fan-out on write and read, use a strategy that minimizes the impact of distributed transactions while ensuring data consistency.

#### Approach
Implement optimistic concurrency control or use eventual consistency models. For example, update follower counts asynchronously to avoid blocking writes.

SQL
-- Asynchronous update for follower count
INSERT INTO Timeline (PostID, UserID, Timestamp, Content, FollowerCount, ProfileName)
VALUES (:new_post_id, :user_id, CURRENT_TIMESTAMP(), 'New Post', 0, :profile_name);

-- Schedule a background job to update followers' counts and notify timelines.
5. Performance Optimization
Indexing: Ensure proper indexing on frequently queried fields like UserID, Timestamp.
Caching: Use distributed caches to store frequently accessed data.
Query Optimization: Optimize queries using EXPLAIN plans to understand execution paths.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, designing a denormalized timeline table involves carefully managing fan-out on write and read. We achieve this by creating materialized views for quick reads, handling fan-out asynchronously where possible, and optimizing queries with proper indexing and caching strategies. Common mistakes include overcomplicating the schema or failing to handle distributed transactions effectively.

#### Follow-ups
could involve discussing specific use cases, trade-offs between different consistency models (strong vs eventual), and how to implement these strategies in a cloud-native environment using services like AWS DynamoDB Streams for real-time updates.

**Example input / output:**

```text
Input: User follows another user.
Output: Update Timeline table and Followers view.

Input: User views their timeline.
Output: Fetch posts from Timeline and Followers tables.
```

**Data:**

```json
{
    "Timeline": {
        "PostID": 1,
        "UserID": 2,
        "Timestamp": "2023-10-05T14:30:00Z",
        "Content": "New Post",
        "FollowerCount": 0,
        "ProfileName": "JohnDoe"
    },
    "Followers": [
        { "FollowerUserID": 1, "FollowedUserCount": 1 }
    ]
}
```

