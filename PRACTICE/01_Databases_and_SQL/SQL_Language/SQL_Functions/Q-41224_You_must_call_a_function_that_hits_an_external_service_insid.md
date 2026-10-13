---
type: "practice"
difficulty: "Deep"
---

# You must call a function that hits an external service inside a query and it hangs the batch. Why are side-effecting functions in the data path an anti-pattern, and what's the fix?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ UDF Performance](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/UDF_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When we place side-effecting functions directly within our SQL queries that interact with external services, we run into several issues such as performance degradation, increased latency, and potential data inconsistencies. These side effects can also cause batch processing to hang or stall unexpectedly. Understanding why these practices are considered an anti-pattern and how to mitigate them is crucial for maintaining robust database systems.

Firstly, let's clarify what a side-effecting function in this context means: any operation that has observable behavior outside of the query itself, such as making HTTP requests, writing files, updating external databases, or even performing complex computations. When these functions are invoked within SQL queries, they can introduce unpredictability and performance bottlenecks.

To reason through this problem from first principles, we need to consider the following assumptions:

The database system is under high load.
The function being called involves a network request, which can be slow or fail.
The external service has varying response times and might not always be available.

Now let's break down the issue step by step:

Step 1: Understanding the Problem

When an SQL query calls an external function that makes an HTTP request to another service, it essentially blocks until the request is completed. This means that every row being processed in the batch will wait for the response from the external service before moving on to the next row. If there are many rows or if the service has frequent outages or slow responses, this can significantly delay the overall processing time and potentially cause the batch to hang indefinitely.

Step 2: Performance Impact

#### Cons
ider a scenario where you have a large dataset with millions of rows and an external function that needs to make an HTTP request for each row. If each request takes even just a few milliseconds, the cumulative latency can quickly add up, causing a noticeable performance drop in your batch processing system.

Step 3: Data Consistency Issues

External functions can also lead to data inconsistencies if they are not idempotent or do not handle errors gracefully. For example, if an external service fails during a request and retries the operation multiple times, it could result in duplicate data entries in your database, leading to inconsistencies.

Step 4: Scalability Challenges

When dealing with high volumes of data, external functions can become a bottleneck. If you have a distributed system where each node processes its own subset of rows, having these nodes wait for an external service response can lead to uneven load distribution and inefficient use of resources.

#### Solution

The key to addressing the issues caused by side-effecting functions in SQL queries is to decouple them from the data path. Here’s how you can achieve this:

Step 1: Use Asynchronous Calls

Instead of making synchronous calls within your query, make asynchronous calls that return immediately without waiting for a response. This allows the database engine to continue processing other rows while the external service handles its request.

SQL
-- Example of an asynchronous call in SQL (pseudo-code)
CREATE FUNCTION async_call_service(input_value) RETURNS void AS $$
BEGIN
    -- Non-blocking, start the process and return immediately
    PERFORM do_async_request(input_value);
END;
$$ LANGUAGE plpgsql;
Step 2: Use External Jobs or Queues

Offload the external service calls to a separate job queue or background processing system. This ensures that your primary batch processing remains unaffected by delays in the external service.

SQL
-- Example of adding a task to a job queue
INSERT INTO job_queue (task_type, input_params) VALUES ('external_service', 'input_value');
Step 3: Batch Processing

Instead of calling the function directly for each row, group rows and process them in batches. This reduces the number of external calls required, improving overall efficiency.

SQL
-- Example of batch processing (pseudo-code)
WITH batched_data AS (
    SELECT * FROM large_dataset LIMIT 1000
)
SELECT async_call_service(input_value) FROM batched_data;
Step 4: Retry Logic and Error Handling

Implement robust retry logic in your background job system to handle temporary failures. This ensures that failed requests are retried automatically without affecting the main processing flow.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The primary anti-pattern of placing side-effecting functions directly within SQL queries is that it ties the database's performance and reliability to external services, leading to unpredictable behavior under high load. By using asynchronous calls, job queues, batch processing, and retry mechanisms, we can mitigate these issues and ensure a more robust and scalable data processing system.

#### Common mistakes
include:

Ignoring the impact of external service delays on batch processing.
Not implementing proper error handling and retries for external functions.
Failing to consider idempotency when designing external interactions.

#### Follow-up
questions might include discussing specific implementation details, trade-offs between different approaches, and best practices for managing concurrency and resource usage in such systems.

**Comparison:**

```
Yes
No
Start
Is function synchronous?
Handle synchronously
Handle asynchronously
Add task to job queue
Process data in batches
Retry on failure
```

**Code:**

```sql
-- Example of an asynchronous call in SQL (pseudo-code)
CREATE FUNCTION async_call_service(input_value) RETURNS void AS $$
BEGIN
    -- Non-blocking, start the process and return immediately
    PERFORM do_async_request(input_value);
END;
$$ LANGUAGE plpgsql;

-- Example of adding a task to a job queue
INSERT INTO job_queue (task_type, input_params) VALUES ('external_service', 'input_value');

-- Example of batch processing (pseudo-code)
WITH batched_data AS (
    SELECT * FROM large_dataset LIMIT 1000
)
SELECT async_call_service(input_value) FROM batched_data;
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `INSERT INTO job_queue (task_type, input_params) VALUES ('external_service', 'input_value');` | Task added to the job queue. |

**Data:**

```json
{
    "job_queue": {
        "task_type": "external_service",
        "input_params": ["data1", "data2"]
    }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is function synchronous?
5. Handle synchronously
6. Handle asynchronously
7. Add task to job queue
8. Process data in batches
9. Retry on failure

