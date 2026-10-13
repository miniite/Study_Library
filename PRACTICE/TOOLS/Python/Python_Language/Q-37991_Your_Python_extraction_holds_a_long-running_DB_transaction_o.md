---
type: "practice"
difficulty: "Deep"
---

# Your Python extraction holds a long-running DB transaction open while writing to S3, blocking other writers. How would you restructure it to shorten the transaction and decouple the write?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Global Quotas Across Requests](<../../../../TOOLS/Python/Python_Language/Global_Quotas_Across_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to refactor our Python code to ensure that long-running database transactions do not block other writers while writing to S3. The key idea is to break down the transaction into smaller parts and use asynchronous I/O operations where possible. This will allow us to write to S3 more frequently without holding up the database connection.

#### Triage
The question probes your ability to handle concurrency issues in Python, specifically focusing on efficient data processing pipelines that integrate with both databases and external storage systems like S3. The primary assumptions are:

The current code uses a long-running transaction for writing to both the database and S3.
We need to optimize this process to reduce the duration of the database transaction while still ensuring reliable writes to S3.

Step-by-Step Solution:

Identify the Bottleneck: Determine if the bottleneck is in reading from the database or writing to S3, or both. This will guide our optimization strategy.
Refactor Transactions: Break down the long-running transaction into smaller chunks that can be processed asynchronously.
Asynchronous I/O for S3 Writes: Use Python’s asyncio and boto3 (Amazon S3 SDK) to write data to S3 asynchronously, allowing multiple writers to run concurrently without blocking each other or the database connection.

#### First-Principles Solution
Let's assume we have a function that currently reads from a database table and writes the results to both an external storage system like S3 and updates a local in-memory cache. We can refactor this as follows:

Data Fetching: Use asyncio to fetch data from the database asynchronously.
Processing and Writing: Process the fetched data locally or store it temporarily, then write parts of the data to S3 using boto3. This ensures that each part is written to S3 as soon as it's ready without blocking further processing.
Database Transaction Management: Ensure that only necessary updates are made in the database transaction and commit them quickly.

Here’s a simplified example:

PYTHON
import asyncio
import boto3

# AWS credentials setup (use environment variables or AWS config file)
s3_client = boto3.client('s3')

async def fetch_data_and_write_to_s3():
    # Asynchronous data fetching from the database
    db_connection = await open_database_connection()

    try:
        async with db_connection.cursor() as cursor:
            await cursor.execute("SELECT * FROM large_table")
            rows = await cursor.fetchall()

            for row in rows:
                # Process and prepare the data to be written to S3
                processed_data = process_row(row)

                # Write to S3 asynchronously
                await write_to_s3(processed_data, s3_client)

    finally:
        db_connection.close()

async def write_to_s3(data, s3_client):
    bucket_name = 'your-bucket-name'
    key = f'output/{uuid.uuid4()}.json'

    response = s3_client.put_object(Bucket=bucket_name, Key=key, Body=json.dumps(data))
    print(f"Written to S3: {key}")

async def main():
    await fetch_data_and_write_to_s3()

# Run the async function
loop = asyncio.get_event_loop()
loop.run_until_complete(main())

This example breaks down the transaction into smaller parts and writes data to S3 asynchronously, reducing the overall transaction time.

#### Closing Summary
+ Common Mistakes/Follow-Ups: By breaking down the database transactions and using asynchronous I/O for S3 operations, we can significantly reduce blocking times. Key considerations include error handling in both async functions, ensuring atomicity of database operations, and managing concurrency issues properly. Always validate data integrity when dealing with distributed systems.

#### Common mistakes
to avoid:

Blocking I/O operations that could be non-blocking.
Not handling errors gracefully in asynchronous code.
Failing to manage resources like database connections efficiently.

**Comparison:**

| Row ID | Name  | Value |
|--------|-------|-------|
| 1      | John  | 100   |
| 2      | Jane  | 200   |
| ...    | ...   | ...   |

**Code:**

```python
import asyncio
import boto3
import uuid
import json

# AWS credentials setup (use environment variables or AWS config file)
s3_client = boto3.client('s3')

async def fetch_data_and_write_to_s3():
    # Asynchronous data fetching from the database
    db_connection = await open_database_connection()

    try:
        async with db_connection.cursor() as cursor:
            await cursor.execute("SELECT * FROM large_table")
            rows = await cursor.fetchall()

            for row in rows:
                # Process and prepare the data to be written to S3
                processed_data = process_row(row)

                # Write to S3 asynchronously
                await write_to_s3(processed_data, s3_client)

    finally:
        db_connection.close()

async def write_to_s3(data, s3_client):
    bucket_name = 'your-bucket-name'
    key = f'output/{uuid.uuid4()}.json'

    response = s3_client.put_object(Bucket=bucket_name, Key=key, Body=json.dumps(data))
    print(f"Written to S3: {key}")

async def main():
    await fetch_data_and_write_to_s3()

# Run the async function
loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```

**Example input / output:**

```

Input:

Database table large_table with multiple rows.
AWS S3 bucket named 'your-bucket-name'.

Output:

Data from each row of large_table is processed and written to a separate file in the S3 bucket.

Example Response:

CODE
Copy
Written to S3: output/1234567890.json
Written to S3: output/9876543210.json
...
```

**Data:**

```json
{
  "large_table": {
    "fields": ["id", "name", "value"],
    "data": [
      {"id": 1, "name": "John", "value": 100},
      {"id": 2, "name": "Jane", "value": 200},
      ...
    ]
  }
}
```

**Flow:**

1. No
2. Yes
3. Fetch Data from Database
4. Is Data Processed?
5. Write to S3 Asynchronously
6. Commit Transaction
7. Process Next Row or End

