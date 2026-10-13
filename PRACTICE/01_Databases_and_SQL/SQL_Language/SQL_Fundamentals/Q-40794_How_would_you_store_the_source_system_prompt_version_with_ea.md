---
type: "practice"
difficulty: "Intermediate"
---

# How would you store the source system prompt version with each response so you can compare answer quality before and after a prompt change?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Experiment Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Experiment_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store the source system prompt version along with each response in an SQL database, we need to ensure that every generated response is associated with its corresponding prompt version. This will allow us to compare answer quality before and after any prompt changes. The core idea is to include a reference to the prompt version in our database schema.

#### Triage

The question is asking about how to store metadata (the source system prompt version) alongside responses in an SQL database for comparative analysis. We need to make assumptions about the data model, such as the format of prompts and responses, and consider any potential performance implications.

#### First-Principles Solution

Database Schema Design:

We will create a table that stores both the response text and its associated prompt version.
This table should include fields for response_id, prompt_version, response_text, and possibly timestamp or date.

#### Example
Table Structure:

SQL
CREATE TABLE responses (
    id INT PRIMARY KEY AUTO_INCREMENT,
    response_id VARCHAR(100) NOT NULL,  -- Unique identifier for the response
    prompt_version VARCHAR(50),         -- Version of the source system prompt used
    response_text TEXT,                 -- The actual generated response text
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP  -- Timestamp when the response was generated
);

Insertion Process:

Every time a new response is generated and stored in the database, we need to include the prompt version associated with that response.

#### Example
SQL insert statement:
SQL
INSERT INTO responses (response_id, prompt_version, response_text)
VALUES ('r123', 'P0.25', 'This is a sample response.');

Querying and Analysis:

To compare the quality of responses before and after a prompt change, we can use SQL queries to filter by prompt_version.

#### Example
query:
SQL
SELECT * FROM responses WHERE prompt_version = 'P0.25';
We can also group results by prompt_version to analyze changes in response quality over time:
SQL
SELECT prompt_version, COUNT(*) as num_responses
FROM responses
GROUP BY prompt_version;

#### Closing Summary
and Common Mistakes
Common Mistake: Forgetting to include the prompt_version field when inserting data.

#### Follow-Up
Discussing how to handle different versions of prompts in a more complex system, possibly using version control systems or tagging mechanisms.

**Code:**

```sql
-- Database schema creation
CREATE TABLE responses (
    id INT PRIMARY KEY AUTO_INCREMENT,
    response_id VARCHAR(100) NOT NULL,
    prompt_version VARCHAR(50),
    response_text TEXT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Example insert statement
INSERT INTO responses (response_id, prompt_version, response_text)
VALUES ('r123', 'P0.25', 'This is a sample response.');

-- Query to select all responses with a specific prompt version
SELECT * FROM responses WHERE prompt_version = 'P0.25';

-- Group by prompt version for analysis
SELECT prompt_version, COUNT(*) as num_responses
FROM responses
GROUP BY prompt_version;
```

**Example input / output:**

```text
INSERT INTO responses (response_id, prompt_version, response_text)
VALUES ('r123', 'P0.25', 'This is a sample response.');

SELECT * FROM responses WHERE prompt_version = 'P0.25';

-- Output:
+----+-----------+------------+-----------------------------+---------------------+
| id | response_id | prompt_version | response_text               | timestamp           |
+----+-----------+------------+-----------------------------+---------------------+
| 1  | r123       | P0.25        | This is a sample response.  | 2023-10-01 14:00:00 |
+----+-----------+------------+-----------------------------+---------------------+

SELECT prompt_version, COUNT(*) as num_responses
FROM responses
GROUP BY prompt_version;

-- Output:
+------------+--------------------+
| prompt_version | num_responses     |
+------------+--------------------+
| P0.25      | 1                  |
+------------+--------------------+
```

