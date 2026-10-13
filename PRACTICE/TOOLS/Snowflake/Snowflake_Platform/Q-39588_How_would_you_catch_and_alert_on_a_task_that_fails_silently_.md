---
type: "practice"
difficulty: "Intermediate"
---

# How would you catch and alert on a task that fails silently because it returns SUCCESS but produced zero rows?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Snowflake Compute and Caching](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Compute_and_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to ensure that tasks in Snowflake, which are designed to return "SUCCESS" even when they don't produce any data (e.g., a SELECT query with no results), actually fail and generate an alert. The key idea is to leverage Snowflake's event listeners or system table captures combined with external alerting mechanisms like Slack or email.

Firstly, let's understand what the question is probing: we need to catch silent failures where a task might complete but produce zero rows, which can lead to misleading success messages and potential issues downstream. The interviewer is likely interested in how we handle such edge cases, our understanding of Snowflake’s event structure, and our ability to integrate external alerting systems.

We will solve this problem by creating an event listener that monitors the SNOWFLAKE.ACCOUNT_USAGE.SQL_EXECUTIONS table for specific conditions (such as zero rows produced) and then triggers an alert. This requires a combination of Snowflake's built-in features, SQL logic, and external integration capabilities.

Step-by-Step Solution
Identify the Condition: We need to monitor the SNOWFLAKE.ACCOUNT_USAGE.SQL_EXECUTIONS table for queries that return zero rows.
Create an Event Listener:
Use Snowflake's event listeners to create a custom alert when certain conditions are met.
Write SQL Logic:
SQL
-- Create a stored procedure to handle the logic
CREATE OR REPLACE PROCEDURE check_zero_rows()
RETURNS VARCHAR
LANGUAGE JAVASCRIPT AS $$
  var sqlText = 'SELECT COUNT(*) FROM YOUR_TABLE WHERE COLUMN_NAME IS NULL';
  var stmt = snowflake.createStatement({sqlText: sqlText});
  var res = stmt.execute();
  while (res.next()) {
    if (res.getColumnValue(1) === 0) {
      return "ALERT";
    }
  }
  return "NO_ALERT";
$$;

-- Create an event listener to trigger the stored procedure
CREATE EVENT LISTENER MY_LISTENER ON DATABASE YOUR_DATABASE
FOR SQL_STATEMENT_SUCCESS, QUERY_END
WHEN CALL check_zero_rows();
Integrate External Alerting:
Set up a webhook or use Snowflake's system table SNOWFLAKE.ACCOUNT_USAGE.EVENTS to send alerts via external services like Slack.
Test and Validate:
Run test queries that return zero rows and ensure the alert is triggered.

#### Closing Summary

In summary, we leveraged Snowflake’s event listener feature combined with a stored procedure to catch silent failures where tasks might complete but produce no data. This ensures that such conditions are detected and alerted upon using external integration tools like Slack or email.

#### Common Mistakes/Follow-Ups

Failing to properly define the condition in SQL.
Not setting up the event listener correctly.
Misconfiguring the external alerting system.
Neglecting edge cases, such as handling different types of queries (SELECT, INSERT, DELETE) that might produce zero rows.

