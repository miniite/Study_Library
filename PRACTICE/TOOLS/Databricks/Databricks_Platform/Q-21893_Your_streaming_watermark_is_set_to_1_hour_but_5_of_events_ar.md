---
type: "practice"
difficulty: "Deep"
---

# Your streaming watermark is set to 1 hour but 5% of events arrive 3 hours late and get dropped. How do you decide between widening the watermark and a separate reconciliation job?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Streaming Ingest Tuning](<../../../../TOOLS/Databricks/Databricks_Platform/Streaming_Ingest_Tuning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where 5% of events are arriving 3 hours late and getting dropped due to the watermark being set at 1 hour, we need to decide between widening the watermark or implementing a separate reconciliation job. This decision involves understanding both the trade-offs and potential impacts on our data pipeline.

First, let’s clarify what the question is really asking:

#### Triage
The interviewer wants to probe your ability to evaluate different solutions for handling late arriving events in a streaming context, considering factors like accuracy, performance, and resource utilization. They are also interested in understanding how you would balance these trade-offs.

#### Assumptions
We assume the watermark is used to determine the freshness of data for processing. Widening it means allowing more time for late events to arrive before they are considered stale. The reconciliation job could involve reprocessing or merging late events into a batch or streaming process.

Now, let’s reason from first principles:

Widening the Watermark:

#### Pros
Simplifies the implementation as you only need to change the watermark configuration.

#### Cons
Widens the retention period of data in memory, potentially increasing memory usage and processing latency. It also might increase the risk of stale data being processed if the late events are not properly managed.

Separate Reconciliation Job:

#### Pros
Ensures that late events are explicitly handled, maintaining data accuracy.

#### Cons
Adds complexity to the pipeline, requiring additional resources and potential performance overhead due to reprocessing or merging steps.

Let’s evaluate both approaches in more detail:

Widening the Watermark

If we widen the watermark from 1 hour to a higher value (e.g., 3 hours), this would allow late events up to 3 hours after the initial event time to be processed. However, it requires careful consideration:

Memory Usage: More data might remain in memory for longer periods.
Latency: Processing could become delayed as more recent data is buffered.
Separate Reconciliation Job

Implementing a separate reconciliation job would involve setting up an additional step or process to handle late events. This could be done in several ways:

Using a Tumbling Window: Process the latest batch of data and then run another pass for any late events.
Event Time Processing: Use event time processing with watermarks but also implement a separate reconciliation mechanism.

Let’s outline both solutions:

Widening the Watermark
PLAINTEXT
1. Set watermark to 3 hours.
2. Ensure that all late events are processed within this extended period.
Separate Reconciliation Job
PLAINTEXT
1. Define a processing pipeline for initial data.
2. Implement a reconciliation step that processes late events separately.
   - Use event time processing with watermarks.
   - Merge or reprocess late events into the main stream.
3. Ensure proper error handling and logging.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In deciding between these options, it's crucial to consider:

Resource Utilization: A wider watermark might consume more memory and processing power.
Data Accuracy: Ensuring that late events are properly handled without compromising the accuracy of real-time data.

#### Common Mistakes

Overlooking the trade-offs: Widening watermarks can lead to increased latency, while separate reconciliation jobs add complexity.
Not considering edge cases: Late events could cause inconsistencies if not managed correctly in both approaches.

#### Follow-ups
might include discussing specific tools and libraries (like Structured Streaming’s window functions), or exploring more advanced strategies like hybrid watermarking techniques that combine the benefits of both methods.

**Comparison:**

| Event ID | Event Time    | Value | Processing Status |
|----------|---------------|-------|-------------------|
| 1        | 2023-10-01T09:00:00Z | A     | Processed         |
| 2        | 2023-10-01T10:00:00Z | B     | Processed         |
| 3        | 2023-10-01T11:00:00Z | C     | Processed         |
| 4        | 2023-10-01T14:00:00Z | D     | Reconciled        |

**Code:**

```python
# Example Python code for setting up a Databricks job with widened watermark
spark = SparkSession.builder \
    .appName("LateEventHandling") \
    .getOrCreate()

df = spark.readStream.format("delta").load("/path/to/delta/table")

query = df.writeStream \
    .outputMode("append") \
    .trigger(processingTime="3 hours") \
    .start("/path/to/output")
```

**Example input / output:**

```text
Input: [event_id, event_time, value]
Output: Processed Data with late events handled

Dry-run:
Initial Data:
1 | 2023-10-01T09:00:00Z | A
2 | 2023-10-01T10:00:00Z | B
3 | 2023-10-01T11:00:00Z | C

Late Events (3 hours late):
4 | 2023-10-01T14:00:00Z | D

After Processing:
1 | 2023-10-01T09:00:00Z | A
2 | 2023-10-01T10:00:00Z | B
3 | 2023-10-01T11:00:00Z | C
4 | 2023-10-01T14:00:00Z | D (handled separately)
```

**Data:**

```json
{
  "watermark": {
    "currentValue": "1 hour",
    "desiredValue": "3 hours"
  },
  "reconciliationJob": {
    "processingStep": [
      "InitialDataProcessing",
      "LateEventReconciliation"
    ]
  }
}
```

**Flow:**

1. FinalOutput
2. LateEventReconciliation
3. InitialDataFlow
4. Data Processing
5. Late Events
6. Initial Data Input
7. Batch Process 1
8. Late Event Reconciliation Job
9. Final Output

