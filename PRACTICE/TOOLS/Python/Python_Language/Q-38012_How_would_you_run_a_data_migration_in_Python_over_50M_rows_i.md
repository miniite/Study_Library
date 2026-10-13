---
type: "practice"
difficulty: "Deep"
---

# How would you run a data migration in Python over 50M rows in production without spiking replication lag or holding long locks, chunking and pacing the work?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Data Migrations in Python](<../../../../TOOLS/Python/Python_Language/Large_Data_Migrations_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate 50 million rows of data efficiently in Python while avoiding high replication lag and long locks, we need to implement a strategy that involves breaking down the task into smaller chunks and pacing the work over time. The goal is to ensure that each chunk is processed without overwhelming the database or causing significant delays.

#### Triage

The core question is how to manage the data migration process effectively in Python while ensuring minimal impact on the production system. We need to consider:

Chunking: Dividing the large dataset into smaller, manageable parts.
Pacing: Processing these chunks at a controlled rate to avoid overwhelming the database.
Lock management: Ensuring that our operations do not hold locks for extended periods.

#### First-Principles Solution
Identify and Configure Database: Ensure your database is configured with appropriate settings such as read replicas or partitioning to handle large volumes of data without impacting performance.
Chunk Data: Split the 50 million rows into smaller chunks, say 10,000-50,000 rows per chunk based on the expected load and available resources.
Asynchronous Processing: Use Python libraries like asyncio or multiprocessing to process these chunks concurrently but control their rate of execution.
Batch Inserts/Updates: Perform batch operations within each chunk to reduce database I/O overhead.
Lock Management: Ensure that transaction locks are managed properly, using techniques like row-level locking where possible.
Detailed Steps

Setup Environment:

PYTHON
import psycopg2  # For PostgreSQL; adjust for other databases as needed
from concurrent.futures import ProcessPoolExecutor
from tqdm import tqdm

connection = psycopg2.connect(database="your_db", user="user", password="password", host="host", port="port")
cursor = connection.cursor()

Chunking and Processing:

Determine the chunk size based on database capacity.
Use a loop to process each chunk in a controlled manner.

Asynchronous Execution:

PYTHON
def process_chunk(chunk_data):
    with connection.cursor() as cursor:
        # Example: Batch insert into target table
        for row in chunk_data:
            sql = "INSERT INTO target_table (column1, column2) VALUES (%s, %s)"
            cursor.execute(sql, row)
        connection.commit()

def migrate_data():
    total_rows = 50_000_000
    chunk_size = 50_000  # Adjust based on database capacity and resources

    for start_idx in tqdm(range(0, total_rows, chunk_size)):
        end_idx = min(start_idx + chunk_size, total_rows)
        chunk_data = fetch_chunk_from_source(start_idx, end_idx)

        with ProcessPoolExecutor() as executor:
            future = executor.submit(process_chunk, chunk_data)
            # Wait for a controlled number of chunks to complete before proceeding
            if (start_idx // chunk_size) % 10 == 0:  # Adjust the interval based on performance monitoring
                futures_to_complete = [future]
                while len(futures_to_complete):
                    done, not_done = concurrent.futures.wait(futures_to_complete)
                    futures_to_complete = not_done

def fetch_chunk_from_source(start_idx, end_idx):
    # Implement logic to fetch chunk data from source database or storage
    pass

migrate_data()

Monitoring and Adjusting:

Monitor the replication lag and adjust chunk size if necessary.
Use tools like pg_stat_activity in PostgreSQL to track lock usage.

#### Closing Summary
and Common Mistakes
Common Mistake 1: Failing to manage locks properly, leading to high contention on the database.
Common Mistake 2: Processing all chunks at once without pacing, causing a spike in replication lag.
Common Mistake 3: Not monitoring performance metrics during the migration process.

#### Conclusion

By carefully chunking and pacing the data migration, managing locks efficiently, and using asynchronous processing techniques, we can ensure that the migration is performed smoothly without overwhelming the production system. Regular monitoring and adjustments based on real-time data will help maintain optimal performance throughout the process.

**Comparison:**

| id | column1  | column2 |
|----|----------|---------|
| 1  | Data1    | Value1  |
| 2  | Data2    | Value2  |

**Code:**

```python
import psycopg2  # For PostgreSQL; adjust for other databases as needed
from concurrent.futures import ProcessPoolExecutor
from tqdm import tqdm

connection = psycopg2.connect(database="your_db", user="user", password="password", host="host", port="port")
cursor = connection.cursor()

def process_chunk(chunk_data):
    with connection.cursor() as cursor:
        for row in chunk_data:
            sql = "INSERT INTO target_table (column1, column2) VALUES (%s, %s)"
            cursor.execute(sql, row)
        connection.commit()

def migrate_data():
    total_rows = 50_000_000
    chunk_size = 50_000

    for start_idx in tqdm(range(0, total_rows, chunk_size)):
        end_idx = min(start_idx + chunk_size, total_rows)
        chunk_data = fetch_chunk_from_source(start_idx, end_idx)

        with ProcessPoolExecutor() as executor:
            future = executor.submit(process_chunk, chunk_data)
            if (start_idx // chunk_size) % 10 == 0:
                futures_to_complete = [future]
                while len(futures_to_complete):
                    done, not_done = concurrent.futures.wait(futures_to_complete)
                    futures_to_complete = not_done

def fetch_chunk_from_source(start_idx, end_idx):
    # Implement logic to fetch chunk data from source database or storage
    pass

migrate_data()
```

**Example input / output:**

```text
# Example input and output
total_rows = 50_000_000
chunk_size = 50_000

start_idx = 0
end_idx = min(start_idx + chunk_size, total_rows)
print(f"Processing chunk from {start_idx} to {end_idx}")

# Simulate fetching data
source_data = [
    (1, "Data1", "Value1"),
    # Add more rows as needed
]

for row in source_data:
    print(f"Processing: {row}")
```

**Data:**

```yaml
# Example schema for source and target tables
source_table:
  columns: [id, column1, column2]

target_table:
  columns: [id, column1, column2]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is start of chunk?
5. Fetch Chunk
6. Process Chunk in Background
7. Increment Start Index
8. End Migration

