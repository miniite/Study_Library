---
type: "practice"
difficulty: "Intermediate"
---

# Your document ingestion for RAG uses a stuck Kafka consumer that silently stops updating the index, so users get answers missing recent docs without any error. What SLI would you put on index freshness and how do you distinguish 'slow' from 'stopped'?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Partition Lag and Hot Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure the document ingestion process in our Retrieval-Augmented Generation (RAG) system stays robust, we need to monitor the freshness of the index while distinguishing between a slow consumer and one that is completely stopped. The primary Service Level Indicator (SLI) I would use for index freshness is the index update rate. This will help us measure how frequently new documents are being indexed.

First, let's break down what "slow" versus "stopped" means in this context:

Slow: The consumer is processing documents but at a pace slower than expected or desired.
Stopped: The consumer has completely halted and is no longer updating the index.

To effectively monitor these states, we will define specific SLIs and thresholds. Here’s how I would approach it:

#### Triaging the Question

The key objective here is to ensure that our document ingestion process remains reliable and responsive. We need to measure not just whether updates are happening but also at what rate. By distinguishing between "slow" and "stopped," we can take appropriate actions to maintain high availability.

#### Assumptions
The Kafka consumer group for ingesting documents into the index is well-configured.
There are no external factors (like network issues) impacting the document ingestion process.
The system has adequate resources (CPU, memory, etc.) to handle the expected load.

#### Solution
from First Principles
Index Update Rate

The primary SLI will be index update rate. This can be measured as follows:

Definition: Number of documents indexed per minute/hour.
Calculation:
Index Update Rate = (Number of New Documents Indexed) / Time Interval

This metric helps us understand the overall performance and speed of document ingestion.

Distinguishing 'Slow' from 'Stopped'

To distinguish between a "slow" consumer and one that is completely stopped, we need to set up monitoring thresholds:

Update Rate Threshold: Define an acceptable update rate based on historical data or service requirements.
Detection Logic:
Slow Consumer Detection: If the Index Update Rate falls below a certain threshold (e.g., 5 documents per minute), flag it as "slow."
Stopped Consumer Detection: If the consumer stops updating for more than X minutes (a configurable timeout), consider it "stopped."

#### Example
Monitoring Setup

Let’s assume we want to monitor every 10 seconds:

Timeout Duration (X): 5 minutes.
Update Rate Threshold: At least 5 documents per minute.

If no updates occur in the last 5 minutes, and the update rate consistently falls below 5 documents/minute over multiple intervals, we would trigger alerts for both "slow" and "stopped" conditions.

#### Closing Summary

By defining the index update rate as our primary SLI, we can effectively monitor the health of document ingestion. The distinction between "slow" and "stopped" consumers will help us take timely actions to maintain system reliability and ensure that users always get up-to-date information from our RAG system.

#### Common Mistakes

Failing to set clear thresholds for what constitutes a slow consumer versus a stopped one.
Not configuring monitoring intervals or detection logic, leading to delayed responses.
Ignoring external factors that might affect the Kafka consumer group, such as network latency or resource constraints.

**Comparison:**

```
Interval	Num New Docs	Update Rate (Docs/Min)
1	3	2.7
2	0	0
3	5	4.6
```

**Code:**

```python
def monitor_index_update_rate(consumer_group):
    # Example code snippet for monitoring index update rate
    update_rate = 0  # Initialize counter
    last_update_time = time.time()

    while True:
        current_time = time.time()

        if current_time - last_update_time >= 10:  # Check every 10 seconds
            num_new_documents = check_num_new_documents()  # Function to count new docs

            if num_new_documents > 0:
                update_rate += num_new_documents
                last_update_time = current_time

        if update_rate / (current_time - start_time) < 5:  # Check for slow consumer
            print("Warning: Consumer is running slowly")

        if time.time() - last_update_time >= 300:  # 5 minute timeout
            print("Alert: Consumer has stopped updating")
```

**Example input / output:**

```

Input:

check_num_new_documents(): A function that returns the number of new documents indexed.

Output:

index_update_rate: The rate at which new documents are being indexed.
slow_consumer_alert (boolean): Indicates if the consumer is running slowly.
stopped_consumer_alert (boolean): Indicates if the consumer has stopped updating.
PYTHON
Copy
check_num_new_documents() -> 3  # Example: 3 new docs in the last interval

# After processing every 10 seconds:
update_rate = 20  # Over 2 minutes, assuming 2 updates of 10 documents each
time_since_last_update = 120  # 2 minutes elapsed

index_update_rate = update_rate / time_since_last_update
if index_update_rate < 5:
    print("Slow Consumer Alert")

if time_since_last_update > 300:  # 5 minute timeout
    print("Stopped Consumer Alert")
```

**Data:**

```json
{
  "update_rate_threshold": 5,
  "timeout_duration_minutes": 5
}
```

**Flow:**

1. Yes
2. < 5 min
3. >= 5 min
4. < 5 docs/minute
5. No
6. Start
7. Is New Doc Count >Threshold?
8. index_update_rate +=num_new_docs
9. Check Time Since Last Update
10. Consumer is Slow
11. Check Update Rate
12. Consumer has StoppedUpdating
13. No Action
14. Send Slow Consumer Alert
15. Send Stopped Consumer Alert

