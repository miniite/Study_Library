---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd data-model a many-to-many social graph (follows/friends) in a wide-column store to support both 'who do I follow' and 'who follows me' efficiently.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Backend Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Search_Backend_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model a many-to-many social graph like "follows" or "friends" in a wide-column store such as Apache Cassandra, we need to focus on both read performance (to answer questions like 'who do I follow' and 'who follows me') and write efficiency. The key is to carefully choose the data model that optimizes for these use cases.

First, let's triage what the question is really asking:

We are designing a schema for a social graph where users can follow multiple other users.
The system needs to support two main queries efficiently: 'who do I follow' and 'who follows me'.
A wide-column store like Apache Cassandra offers flexibility in storing and querying data, but we need to ensure that our model leverages this effectively.

#### Assumptions

We have a large user base with frequent updates (follow/unfollow) and reads.
The system should support both real-time and batch processing scenarios.

Step-by-step from first principles:

Identify the entities: Users and Follows relationships.

Determine the queries:

'who do I follow': Given a user, return all users they are following.
'who follows me': Given a user, return all users who are following them.

Design the data model:

We will use two tables in Cassandra: one for storing follow relationships and another for tracking follower counts.
Data Model
Follow Table (for storing the actual follows):
CQL
CREATE TABLE IF NOT EXISTS Follows (
    follower_id UUID,  // The ID of the user following someone
    followed_id UUID,  // The ID of the user being followed
    PRIMARY KEY ((follower_id), followed_id)
) WITH CLUSTERING ORDER BY (followed_id DESC);
Followers Table (for tracking how many users follow a given user):
CQL
CREATE TABLE IF NOT EXISTS Followers (
    followed_id UUID,  // The ID of the user being followed
    follower_count BIGINT,  // The number of followers this user has
    PRIMARY KEY (followed_id)
) WITH CLUSTERING ORDER BY (follower_count DESC);
Querying the Data
Who do I follow:
CQL
SELECT * FROM Follows WHERE follower_id = <user_id>;
Who follows me:
CQL
SELECT followed_id, follower_count FROM Followers WHERE followed_id = <user_id>;
Handling Updates and Deletes

When a user unfollows another user:

Delete the corresponding row in the Follows table.
Decrease the follower_count by one in the Followers table.

When a new follow is created:

Insert a new row in the Follows table.
Increase the follower_count in the Followers table for the followed user.

#### Performance Considerations
Indexing: Ensure that the primary key and clustering columns are chosen to optimize read performance.
Partition Key: Use the follower_id as the partition key in the Follows table to balance reads across nodes.

#### Cons
istency Level: Choose an appropriate consistency level for your use case, considering the trade-off between availability and data accuracy.

#### Closing Summary

This model ensures that both 'who do I follow' and 'who follows me' queries are efficient by leveraging Cassandra's wide-column capabilities. The Follows table handles bidirectional relationships efficiently, while the Followers table tracks follower counts for quick read operations.

#### Common Mistakes/Follow-Ups

Overusing In-Memory Storage: Ensure that in-memory storage (like Redis) is not overused as it can lead to data inconsistencies.
Incorrect Partition Key Selection: Poor selection of partition keys can lead to hotspotting and uneven distribution of reads/writes across nodes.
Ignoring Write Performance: Ensure writes are optimized for performance, especially during high-volume follow/unfollow events.

