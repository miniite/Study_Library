---
type: "practice"
difficulty: "Intermediate"
---

# In a Q&A system how would you model voting and compute a user's reputation?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Threaded Comments Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Threaded_Comments_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a low-level data model for voting in a Q&A system, our primary goal is to ensure that the system can efficiently handle large volumes of votes while accurately calculating users' reputations based on these votes. The key aspects include modeling the relationships between users, questions, answers, and votes.

#### Triage
Understanding the Question

The question is asking for a detailed design of how voting mechanisms are modeled in a Q&A system, specifically focusing on computing user reputation. We need to consider:

How votes (upvotes/downvotes) are stored.
Relationships between users, questions, and answers.
How to efficiently calculate user reputations based on these votes.

#### First-Principles Solution
Modeling Votes:
A vote can be related to a specific answer by a user.
We need to store the timestamp of when the vote was cast for auditing purposes.
Relationships:
Users can cast multiple votes on different answers.
Calculating Reputation:
The reputation of a user is calculated based on the aggregate votes they receive (both as voters and answerers).
Step-by-Step Solution

Define the Vote Table:

votes table with columns:
vote_id: Unique identifier for each vote.
user_id: ID of the user who cast the vote.
answer_id: ID of the answer to which the vote was cast.
vote_type: Enum value indicating whether it's an upvote or downvote (e.g., 'UPVOTE', 'DOWNVOTE').
timestamp: Timestamp when the vote was cast.

Ensure Referential Integrity:

The user_id and answer_id fields should reference the corresponding users and answers in their respective tables.

Aggregate Votes to Calculate Reputation:

For each user, sum up all positive (upvotes) and negative (downvotes) votes received.

#### Closing Summary

To model voting and compute a user's reputation in a Q&A system, we need to establish a clear structure for storing votes, ensure referential integrity, and implement an efficient way to calculate reputation. By following these steps, we can build a robust system that handles high volumes of data while maintaining accuracy.

#### Common Mistakes/Follow-Ups
Incorrect Vote Type Handling: Failing to properly track vote types (upvote/downvote) can lead to incorrect reputation calculations.
Duplicate Voting: Allowing users to change their votes without proper handling can result in inaccurate reputation scores.

#### Performance Considerations
Large numbers of votes can impact performance if not optimized, such as by using efficient indexing and caching mechanisms.

**Comparison:**

| users       | answers        | votes         |
|-------------|----------------|---------------|
| user_id     | username, email | answer_id     |
|            |                | question_id   |
|            |                | content       |
|            |                | created_at    |
| vote_id     | user_id        | timestamp     |
|            | answer_id      | vote_type     |

**Code:**

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(100),
    email VARCHAR(150)
);

CREATE TABLE answers (
    answer_id INT PRIMARY KEY,
    question_id INT,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE votes (
    vote_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    answer_id INT,
    vote_type ENUM('UPVOTE', 'DOWNVOTE') NOT NULL,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (answer_id) REFERENCES answers(answer_id)
);
```

**Example input / output:**

```text
-- Example Data
INSERT INTO users (user_id, username, email) VALUES
(1, 'alice', 'alice@example.com'),
(2, 'bob', 'bob@example.com');

INSERT INTO answers (answer_id, question_id, content) VALUES
(101, 5, 'This is a great answer!'),
(102, 5, 'Another good one here.');

INSERT INTO votes (user_id, answer_id, vote_type) VALUES
(1, 101, 'UPVOTE'),
(1, 101, 'UPVOTE'),
(2, 101, 'DOWNVOTE'),
(2, 102, 'UPVOTE');

-- Calculate Reputation for User 1
SELECT SUM(CASE WHEN vote_type = 'UPVOTE' THEN 1 ELSE -1 END) AS reputation FROM votes WHERE user_id = 1;
```

**Flow:**

1. User
2. Votes on Answers
3. Vote Details(Upvote/Downvote)
4. Calculate Reputation Based onVotes
5. Answer Creation and Updates

