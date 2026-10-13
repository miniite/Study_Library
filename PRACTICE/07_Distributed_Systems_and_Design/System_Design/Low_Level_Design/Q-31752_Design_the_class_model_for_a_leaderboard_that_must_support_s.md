---
type: "practice"
difficulty: "Deep"
---

# Design the class model for a leaderboard that must support seasonal resets, ties broken by earliest-achieved, and 'friends-only' views; how do you structure storage so all three queries stay fast?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Threaded Comments Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Threaded_Comments_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing the class model for a leaderboard with seasonal resets, tie-breaking based on the earliest achievement date, and friends-only views, we need to ensure that our data structures support these functionalities efficiently. The key challenge is maintaining performance across multiple query types while minimizing storage overhead.

The question can be broken down into several core components:

Seasonal Resets: Each season should have its own leaderboard.
Tie-Breaking by Earliest Achievement Date: Users with the same score should be ranked based on when they achieved their scores first.
Friends-Only Views: A user can view only their friends' scores, which requires a way to establish friendships between users.

To achieve this, we need to design our classes and storage schema carefully:

User Model: Each user will have an ID, a name, and a list of friend IDs.
Score Entry Model: Each score entry will include the user ID, the score, and the achievement date.
Seasonal Leaderboard Model: A leaderboard will be tied to a specific season and maintain its own set of score entries.

Let's dive into the design from first principles:

Step-by-Step Design

User Class:

userId: Unique identifier for each user.
name: Name of the user.
friends: A list of friend IDs to establish friendships between users.

ScoreEntry Class:

scoreId: Unique identifier for each score entry.
userId: The ID of the user who achieved this score.
score: The score value.
achievementDate: The date when the score was achieved.

SeasonalLeaderboard Class:

seasonId: Identifier for the current season.
users: A map from userId to a list of ScoreEntry objects, ensuring we can quickly retrieve and sort entries by score and achievement date.
Data Structure

For storage, we will use SQL databases like PostgreSQL or NoSQL databases like MongoDB. The choice depends on our exact requirements for querying speed and consistency. Here's an example schema using PostgreSQL:

SQL
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    friends TEXT[] -- An array of friend IDs
);

CREATE TABLE score_entries (
    score_id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users (user_id) ON DELETE CASCADE,
    score INTEGER NOT NULL,
    achievement_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Querying the Leaderboard
Seasonal Reset: To reset the leaderboard for a specific season, we can simply truncate or delete entries from that season’s table.
Tie-Breaking by Earliest Achievement Date: Using an SQL query, we can retrieve and sort score entries based on score and achievement_date.
SQL
SELECT user_id, AVG(score) as average_score, MIN(achievement_date) as earliest_date
FROM score_entries
WHERE season_id = <current_season>
GROUP BY user_id
ORDER BY average_score DESC, earliest_date ASC;
Friends-Only View: To provide a friends-only view for a specific user:
SQL
SELECT user_id, AVG(score) as average_score, MIN(achievement_date) as earliest_date
FROM score_entries
WHERE season_id = <current_season> AND user_id IN (SELECT * FROM users WHERE user_id IN <friends_list>)
GROUP BY user_id
ORDER BY average_score DESC, earliest_date ASC;

#### Closing Summary

By structuring our classes and storage models carefully, we can support all three critical functionalities: seasonal resets, tie-breaking by the earliest achievement date, and friends-only views. The key is using appropriate data structures (like maps in SQL) to ensure fast retrieval and sorting of score entries.

#### Common Mistakes/Follow-Ups
Overcomplicating the Storage Schema: Avoid creating unnecessary tables or indexes that do not contribute to query performance.
Ignoring Scalability: Ensure your database design can handle a large number of users and scores, especially during peak times like new seasons starting.
Not Considering Indexing: Proper indexing on user_id and season_id will significantly improve query performance.

**Comparison:**

```
User
Score Entry
Seasonal Leaderboard
```

**Code:**

```sql
-- PostgreSQL User Table
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    friends TEXT[] -- An array of friend IDs
);

-- PostgreSQL Score Entry Table
CREATE TABLE score_entries (
    score_id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users (user_id) ON DELETE CASCADE,
    score INTEGER NOT NULL,
    achievement_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Example input / output:**

```text
-- Query for friends-only view of Alice's season leaderboard
SELECT user_id, AVG(score) as average_score, MIN(achievement_date) as earliest_date
FROM score_entries
WHERE season_id = 2 AND user_id IN (1, 2)
GROUP BY user_id
ORDER BY average_score DESC, earliest_date ASC;
```

**Data:**

```json
{
  "users": [
    { "userId": 1, "name": "Alice", "friends": [2, 3] },
    { "userId": 2, "name": "Bob", "friends": [] },
    { "userId": 3, "name": "Charlie", "friends": [4] }
  ],
  "scoreEntries": [
    { "scoreId": 1, "userId": 1, "score": 100, "achievementDate": "2023-10-01" },
    { "scoreId": 2, "userId": 1, "score": 95, "achievementDate": "2023-08-01" },
    { "scoreId": 3, "userId": 2, "score": 100, "achievementDate": "2023-07-01" },
    { "scoreId": 4, "userId": 3, "score": 95, "achievementDate": "2023-06-01" }
  ]
}
```

**Flow:**

1. = seasonal_leaderboard_flowchart =
2. graph TD
3. User-->|Insert ScoreEntry| ScoreEntry
4. ScoreEntry-->|Group By Season, User| GroupedScores
5. GroupedScores-->|Sort by Avg Score & Earliest Date| SortedScores
6. SortedScores-->|Return Top N Users| TopUsers

