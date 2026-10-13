---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to find conversations that triggered a moderation flag and count how many distinct users they came from?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conditional Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Conditional_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to write an SQL query that identifies conversations which have been flagged for moderation and counts the number of distinct users involved in these flagged conversations. We'll start by breaking down what the problem is asking and then build our solution step-by-step.

#### Triaging the Question

The core of the question involves:

Identifying conversations: This means we need to consider a table that stores conversation data.
Moderation flags: There should be another table where moderation flags are stored, linking back to the conversations.
Distinct users: We need to count unique users from these flagged conversations.

#### Assumptions
Assume there is a conversations table with columns like conversation_id, user_id, etc.
Assume there is a moderation_flags table that contains flags for each conversation, possibly with a column flagged_conversation_id.
Step-by-Step Solution
Identify Flagged Conversations: We need to find the conversations that have been flagged.
Join Tables: Join the conversations and moderation_flags tables on their respective IDs.
Count Distinct Users: Use a GROUP BY clause with a COUNT(DISTINCT user_id) function.

Here's how we can write this query:

SQL
SELECT
    COUNT(DISTINCT c.user_id) AS distinct_users_count,
    m.flagged_conversation_id
FROM
    conversations c
JOIN
    moderation_flags mf ON c.conversation_id = mf.flagged_conversation_id
GROUP BY
    m.flagged_conversation_id;
Explanation of the Query:
Step 1: We select a count of distinct user_ids that are involved in flagged conversations.
Step 2: The query joins the conversations table (c) with the moderation_flags table (mf) on the condition that their conversation IDs match.
Step 3: We group by the flagged_conversation_id to ensure we count distinct users for each flagged conversation.

#### Closing Summary

This SQL query effectively identifies and counts the number of distinct users involved in conversations that have been flagged for moderation. It ensures that we only include unique user IDs per flagged conversation, providing a clear and concise result.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to use COUNT(DISTINCT) which would not correctly count the number of distinct users.

#### Follow-Up
If there's a need for more detailed information about these conversations (e.g., specific details in the conversations table), we might add additional columns or join with other tables.

**Code:**

```sql
SELECT
    COUNT(DISTINCT c.user_id) AS distinct_users_count,
    m.flagged_conversation_id
FROM
    conversations c
JOIN
    moderation_flags mf ON c.conversation_id = mf.flagged_conversation_id
GROUP BY
    m.flagged_conversation_id;
```

