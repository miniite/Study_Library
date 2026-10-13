---
type: "practice"
difficulty: "Intermediate"
---

# You want to know how much of a table's data is actually in shared_buffers to explain cache-miss latency; how would you use the pg_buffercache extension and what does the hit ratio tell you?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Finding Expensive Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Finding_Expensive_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine how much of a PostgreSQL table’s data is currently cached in shared_buffers, we can leverage the pg_buffercache extension. This tool provides detailed information about which blocks are loaded into memory and where they reside (on disk or in cache). Specifically, we're interested in calculating the hit ratio to understand cache-miss latency.

#### Triage

The question is asking how to use the pg_buffercache extension to gauge the cache hit rate of a PostgreSQL table. The core idea here is to compare the number of cached blocks with the total number of blocks accessed. This will give us an insight into how much data is already in memory and potentially reduce cache-miss latency.

#### Assumptions
You have pg_buffercache installed and enabled.
You are familiar with basic PostgreSQL queries and operations.
The table in question has a significant amount of data to provide meaningful insights.

#### Solution

First, let’s start by querying the pg_buffercache extension:

SQL
SELECT
    nmspname AS schema_name,
    relname AS table_name,
    blckno::text AS block_number,
    age(blcktimestamp) as last_accessed_time
FROM
    pg_buffercache
WHERE
    relname = 'your_table_name';

Replace 'your_table_name' with the name of the table you are interested in. This will give us a list of blocks that are currently cached, along with their last access times.

To get the total number of data blocks for the table, we can use:

SQL
SELECT
    pg_total_relation_size('your_table_name') / 8192 AS total_blocks;

This calculates the size of the table in pages (assuming each page is 8KB).

Next, to calculate the hit ratio, you would need to count the number of cached blocks and compare it with the total number of blocks accessed. A common approach is to periodically run a workload that simulates read operations on the table, then check the pg_buffercache output after each operation.

#### Example
Workflow
Run a workload that reads from the table.
After running the workload, query pg_buffercache:
SQL
SELECT
    nmspname AS schema_name,
    relname AS table_name,
    blckno::text AS block_number,
    age(blcktimestamp) as last_accessed_time
FROM
    pg_buffercache
WHERE
    relname = 'your_table_name';
Count the number of cached blocks.
Calculate the hit ratio:
SQL
WITH buffercache_data AS (
    SELECT
        nmspname AS schema_name,
        relname AS table_name,
        blckno::text AS block_number,
        age(blcktimestamp) as last_accessed_time
    FROM
        pg_buffercache
    WHERE
        relname = 'your_table_name'
)
SELECT
    COUNT(*) / (SELECT pg_total_relation_size('your_table_name') / 8192) AS hit_ratio
FROM
    buffercache_data;

#### Closing Summary
and Common Mistakes

Using pg_buffercache effectively requires understanding both the extension's capabilities and PostgreSQL’s internal caching mechanisms. The common mistakes include not running a representative workload, failing to account for all block types (e.g., index blocks), or misinterpreting the cache hit ratio.

Always ensure that your test scenario accurately reflects real-world usage patterns to get meaningful results.

Common Follow-Ups
How do you handle tables with indexes?
What happens if shared_buffers is too small?
Can we optimize the cache using other techniques?

**Comparison:**

```
Schema Name	Table Name	Block Number	Last Accessed Time
public	example_table	1234	8 days ago
public	example_table	5678	3 hours ago
```

**Code:**

```sql
-- Query to check cached blocks and their last access times
SELECT
    nmspname AS schema_name,
    relname AS table_name,
    blckno::text AS block_number,
    age(blcktimestamp) as last_accessed_time
FROM
    pg_buffercache
WHERE
    relname = 'your_table_name';

-- Calculate total blocks in the table
SELECT
    pg_total_relation_size('your_table_name') / 8192 AS total_blocks;

-- Calculate hit ratio
WITH buffercache_data AS (
    SELECT
        nmspname AS schema_name,
        relname AS table_name,
        blckno::text AS block_number,
        age(blcktimestamp) as last_accessed_time
    FROM
        pg_buffercache
    WHERE
        relname = 'your_table_name'
)
SELECT
    COUNT(*) / (SELECT pg_total_relation_size('your_table_name') / 8192) AS hit_ratio
FROM
    buffercache_data;
```

**Example input / output:**

```text
-- Input:
WITH buffercache_data AS (
    SELECT
        nmspname AS schema_name,
        relname AS table_name,
        blckno::text AS block_number,
        age(blcktimestamp) as last_accessed_time
    FROM
        pg_buffercache
    WHERE
        relname = 'example_table'
)
SELECT
    COUNT(*) / (SELECT pg_total_relation_size('example_table') / 8192) AS hit_ratio
FROM
    buffercache_data;

-- Output:
hit_ratio: 0.75
```

**Data:**

```json
{
  "table_name": "example_table",
  "total_blocks": 4096,
  "cached_blocks": 3072
}
```

**Flow:**

1. Start
2. Run workload on table
3. RUN pg_buffercache
4. Count cached blocks
5. Calculate hit ratio
6. Analyze results
7. End

