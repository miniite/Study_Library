---
type: "practice"
difficulty: "Intermediate"
---

# How would you instrument a cursor-based procedure to capture per-iteration timing so you can find which fetch or DML is the hotspot?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Bulk Collect and FORALL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Bulk_Collect_and_FORALL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To instrument a cursor-based procedure and capture per-iteration timing, we need to understand the core issue: identifying performance bottlenecks in specific parts of the iteration loop. This involves logging timestamps at key points within each iteration so that we can analyze which fetch or DML operations are taking the most time.

#### Triage

The question is asking for a method to instrument a cursor-based procedure such that timing data is captured and logged per-iteration, allowing us to pinpoint where the execution time is being spent. The interviewer may be probing your understanding of database cursors, procedural programming in SQL, and performance optimization techniques. We need to make assumptions about the existing cursor-based procedure's structure and how we can integrate new logging without disrupting its functionality.

#### First-Principles Solution
Identify Key Points: Determine where precise timing is needed—typically at the start of each fetch and any DML operations.
Integrate Timing Logic: Embed DBMS_SESSION.SLEEP or equivalent to simulate a low-level timer, or use SQL timestamps to record the current time.
Logging Mechanism: Use logging mechanisms such as writing to a trace file or a database table to store timing data.
Detailed Steps

Before Each Iteration:

Capture the start time of the iteration.

Before Fetch Operation:

Log the elapsed time since the start of the procedure and before the fetch operation.

After Fetch Operation:

Log the duration taken by the fetch operation.

Before DML Operations:

Log the duration from the end of the last iteration to the beginning of the current DML operation.

After DML Operation:

Log the duration for the DML operation itself.

End of Iteration:

Log any cleanup or post-operation tasks, such as releasing locks if necessary.

#### Example
Code
SQL
DECLARE
  v_start_time TIMESTAMP;
  v_fetch_end_time TIMESTAMP;
  v_dml_end_time TIMESTAMP;
BEGIN
  -- Capture start time before the cursor loop
  v_start_time := SYSTIMESTAMP;

  FOR rec IN (SELECT ... FROM table_name) LOOP
    -- Log timing of fetch operation
    DBMS_OUTPUT.PUT_LINE('Fetch Time: ' || TO_CHAR(SYSTIMESTAMP - v_start_time));

    -- Perform DML operations on the fetched record
    UPDATE some_table SET col1 = val WHERE id = rec.id;
    -- Capture end time after update
    v_dml_end_time := SYSTIMESTAMP;

    DBMS_OUTPUT.PUT_LINE('Update Time: ' || TO_CHAR(v_dml_end_time - SYSTIMESTAMP));
  END LOOP;

  -- Clean up or perform any final actions
END;

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that we capture detailed timing information per-iteration, which helps in identifying the performance hotspots. The main mistake to avoid is overcomplicating the code within each iteration, which can lead to significant overhead. Instead, focus on capturing essential timings.

Common follow-ups might include how to handle large datasets, potential concurrency issues, and what additional metrics could be useful (e.g., memory usage).

**Code:**

```sql
DECLARE
  v_start_time TIMESTAMP;
  v_fetch_end_time TIMESTAMP;
  v_dml_end_time TIMESTAMP;
BEGIN
  -- Capture start time before the cursor loop
  v_start_time := SYSTIMESTAMP;

  FOR rec IN (SELECT ... FROM table_name) LOOP
    -- Log timing of fetch operation
    DBMS_OUTPUT.PUT_LINE('Fetch Time: ' || TO_CHAR(SYSTIMESTAMP - v_start_time));

    -- Perform DML operations on the fetched record
    UPDATE some_table SET col1 = val WHERE id = rec.id;
    -- Capture end time after update
    v_dml_end_time := SYSTIMESTAMP;

    DBMS_OUTPUT.PUT_LINE('Update Time: ' || TO_CHAR(v_dml_end_time - SYSTIMESTAMP));
  END LOOP;

  -- Clean up or perform any final actions
END;
```

