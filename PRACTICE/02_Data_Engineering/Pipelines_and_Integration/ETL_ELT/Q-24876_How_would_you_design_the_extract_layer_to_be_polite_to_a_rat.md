---
type: "practice"
difficulty: "Intermediate"
---

# How would you design the extract layer to be polite to a rate-limited third-party API while still ingesting millions of records a day?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ ETL vs ELT Design](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/ETL_vs_ELT_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an extract layer that is both polite and efficient in handling data from a rate-limited third-party API, we need to focus on two main aspects: adhering to the API's rate limits and optimizing data ingestion processes. The key idea here is to implement a strategy that respects the API's constraints while ensuring continuous and robust data collection.

The question is probing our understanding of how to manage external dependencies effectively in an ETL (Extract, Transform, Load) or ELT (Extract, Load, Transform) pipeline. We need to consider the rate limits imposed by the third-party API, ensure that we do not exceed these limits, and still meet our data ingestion goals.

#### Triaging the Question
Rate Limits: The third-party API has a maximum number of requests per unit time.
Volume of Data: Millions of records need to be ingested daily.
Politeness: We must avoid overwhelming the API with too many requests, which could lead to IP blocking or other penalties.

#### Assumptions
The rate limit is known and documented by the third-party provider.
The data can be divided into smaller chunks that fit within the rate limit constraints.
There are no severe latency issues or dependencies on real-time processing for this data.
Step-by-Step Solution from First Principles
1. Understanding Rate Limits

First, we need to understand and document the rate limits provided by the third-party API. Suppose the provider allows 100 requests per minute with a maximum of 5000 records per request. This means we can make up to 500,000 records per hour.

2. Chunking Data

To respect the rate limit, we can split our data ingestion process into smaller chunks that fit within the allowable limits. For example, if we need to ingest 1 million records daily, we can break this down into 24-hour blocks, each containing approximately 41,667 records.

3. Implementing a Queue System

We will use a message queue (e.g., RabbitMQ, Kafka) to manage the ingestion process. This helps in managing backpressure and allows us to handle failures gracefully. Here’s how we can structure it:

Producer: Sends chunks of data to the queue.

#### Cons
umer(s): Processes these chunks at an appropriate rate.
4. Rate Limiting with Queues

Each consumer should be configured to adhere to the rate limit. For example, if the API allows 100 requests per minute and each request processes up to 5000 records, then each consumer can handle up to 5000 records every 6 seconds.

5. Backoff Strategy

If a request fails due to hitting the rate limit or another reason, we need a backoff strategy:

Exponential Backoff: Wait for an increasing amount of time before retrying (e.g., wait 1 second after the first failure, 2 seconds after the second, and so on).
Retry Logic: Implement exponential retries with a maximum number of attempts.
6. Monitoring and Logging

Set up monitoring to track API usage, response times, and any issues that arise during data ingestion. Logs should capture errors and provide insights into performance bottlenecks.

#### Closing Summary

By implementing a queue-based system and adhering strictly to rate limits, we can ensure that the extract layer is both polite to the third-party API and efficient in ingesting large volumes of data. This approach balances respect for external constraints with robustness in our data pipeline.

#### Common Mistakes/Follow-Ups
Overlooking Rate Limits: Ignoring the documented rate limits can lead to IP blocking or other penalties.
Insufficient Error Handling: Not implementing a proper backoff strategy and retry logic can result in failed ingestions.
Lack of Monitoring: Without adequate monitoring, issues may go unnoticed, leading to potential data loss or delayed processing.

**Comparison:**

```
Time	Actions
0	Start
6s	Request 1/100 sent, process first chunk of 41667 records
12s	Request 2/100 sent, process second chunk of 41667 records
```

**Code:**

```python
import time
from queue import Queue

class RateLimitedConsumer:
    def __init__(self, max_requests_per_minute=100, records_per_request=5000):
        self.max_requests_per_minute = max_requests_per_minute
        self.records_per_request = records_per_request
        self.request_count = 0
        self.next_request_time = time.time()

    def process_chunk(self, chunk):
        current_time = time.time()
        if (current_time - self.next_request_time) < 60 / self.max_requests_per_minute:
            wait_time = (1 + (self.next_request_time + 60 / self.max_requests_per_minute) - current_time)
            time.sleep(wait_time)

        # Process the chunk
        for record in chunk:
            print(f"Processing record: {record}")

        self.request_count += len(chunk)
        if self.request_count >= self.records_per_request:
            self.next_request_time = time.time() + 60 / self.max_requests_per_minute - (current_time - self.next_request_time)
            self.request_count = 0
```

**Example input / output:**

```text
Input:
- A queue with chunks of data (e.g., [record1, record2, ...])

Output:
- Processed records in a polite manner (no more than 5000 records per request every 6 seconds)
```

**Data:**

```json
{
  "rate_limit": {
    "max_requests_per_minute": 100,
    "records_per_request": 5000
  },
  "chunk_size": 41667,
  "total_records": 1000000
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is request count >= maxrequests?
5. Wait for next minute
6. Process chunk and resetcounter

