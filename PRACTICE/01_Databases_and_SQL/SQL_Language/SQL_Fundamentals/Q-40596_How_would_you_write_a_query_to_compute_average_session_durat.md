---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to compute average session duration per user, handling sessions with a NULL end time?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Retention Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Retention_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

SQL
-- Sample Input Data

CREATE TABLE Sessions (
    userId INT,
    sessionId INT PRIMARY KEY,
    startTime TIMESTAMP,
    endTime TIMESTAMP
);

INSERT INTO Sessions (userId, sessionId, startTime, endTime) VALUES
(1, 101, '2023-09-10 14:00:00', '2023-09-10 15:00:00'),
(1, 102, '2023-09-10 16:00:00', NULL),
(2, 201, '2023-09-11 08:00:00', '2023-09-11 10:00:00'),
(2, 202, '2023-09-11 14:00:00', NULL);
SQL
-- SQL Query to Compute Average Session Duration per User

SELECT
    userId,
    AVG(
        CASE
            WHEN endTime IS NOT NULL THEN (endTime - startTime)
            ELSE (CURRENT_TIMESTAMP - startTime)
        END
    ) AS avg_session_duration
FROM Sessions
GROUP BY userId;

