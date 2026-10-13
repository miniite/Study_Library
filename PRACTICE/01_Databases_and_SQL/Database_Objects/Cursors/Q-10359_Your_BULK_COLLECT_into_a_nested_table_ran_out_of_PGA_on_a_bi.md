---
type: "practice"
difficulty: "Expert"
---

# Your BULK COLLECT into a nested table ran out of PGA on a big result — how would you cap memory while keeping throughput acceptable?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Bulk Collect and FORALL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Bulk_Collect_and_FORALL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where BULK COLLECT into a nested table exceeds PGA memory, we need to find a balance between maximizing throughput and maintaining manageable memory usage. This can be achieved by leveraging cursors in PL/SQL with careful buffer management. Let's break down the solution step-by-step.

Firstly, understand that the problem arises because the BULK COLLECT operation tries to load all rows at once into an array, which consumes significant PGA (Program Global Area) memory if the result set is large. The goal here is to process data in chunks instead of all at once.

Secondly, let's consider the assumptions:

We are dealing with a large dataset that exceeds available PGA memory.
We need to keep throughput acceptable while minimizing PGA usage.
We have access to PL/SQL programming.

To achieve this, we can use a cursor to fetch data in smaller batches and process each batch separately. Here’s how you would implement it:

Define the Cursor: Use an explicit cursor to define the query that will be executed against the database.
Fetch Rows in Chunks: Instead of fetching all rows at once, we'll fetch a manageable number of rows per iteration.
Process Each Batch: Process each batch of fetched rows individually before moving on to the next chunk.

Here’s an example of how you can implement this:

SQL
DECLARE
  -- Define cursor for selecting data in chunks
  CURSOR c_data IS
    SELECT column1, column2, ...
      FROM your_table
     WHERE some_condition;

  -- Declare variables to hold the fetched rows and their count
  TYPE t_row_type IS RECORD (
    col1 VARCHAR2(100),
    col2 NUMBER,
    -- Add other columns as needed
  );

  TYPE t_rows_table IS TABLE OF t_row_type;
  v_rows t_rows_table := t_rows_table();

  v_chunk_size CONSTANT PLS_INTEGER := 1000; -- Adjust based on your memory and throughput requirements
BEGIN
  -- Open the cursor to start fetching data
  OPEN c_data;

  LOOP
    FETCH c_data BULK COLLECT INTO v_rows LIMIT v_chunk_size;

    -- Process each batch of rows
    FOR i IN 1..v_rows.COUNT LOOP
      -- Perform your operations on each row here
      DBMS_OUTPUT.PUT_LINE('Processing: ' || v_rows(i).col1);
    END LOOP;

    EXIT WHEN v_rows.COUNT = 0; -- Stop when no more rows are fetched

    -- Clear the table for next iteration if necessary
    v_rows := t_rows_table();

    -- Optionally, adjust v_chunk_size based on performance metrics or other factors
  END LOOP;

  -- Close the cursor to free up resources
  CLOSE c_data;
END;

This script fetches rows in chunks of size v_chunk_size and processes each chunk before moving to the next. This way, you avoid overwhelming PGA memory while ensuring high throughput.

**Code:**

```sql
DECLARE
  CURSOR c_data IS
    SELECT column1, column2, ...
      FROM your_table
     WHERE some_condition;

  TYPE t_row_type IS RECORD (
    col1 VARCHAR2(100),
    col2 NUMBER,
    -- Add other columns as needed
  );

  TYPE t_rows_table IS TABLE OF t_row_type;
  v_rows t_rows_table := t_rows_table();

  v_chunk_size CONSTANT PLS_INTEGER := 1000; -- Adjust based on your memory and throughput requirements
BEGIN
  OPEN c_data;

  LOOP
    FETCH c_data BULK COLLECT INTO v_rows LIMIT v_chunk_size;

    FOR i IN 1..v_rows.COUNT LOOP
      DBMS_OUTPUT.PUT_LINE('Processing: ' || v_rows(i).col1);
    END LOOP;

    EXIT WHEN v_rows.COUNT = 0;

    v_rows := t_rows_table();
  END LOOP;

  CLOSE c_data;
END;
```

