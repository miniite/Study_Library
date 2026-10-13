---
type: "practice"
difficulty: "Deep"
---

# How would you set a platform strategy for handling late-arriving and out-of-order data as a cross-cutting concern rather than each pipeline solving it inconsistently — watermarks, restatement, or SCD at the platform layer?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Late and Out-of-order Data Strategy](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Late_and_Out-of-order_Data_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In setting a platform strategy to handle late-arriving and out-of-order data as a cross-cutting concern rather than each pipeline solving it inconsistently, we need to focus on consistency, scalability, and robustness at the platform level. This involves strategies like watermarks, restatement mechanisms, or Slowly Changing Dimensions (SCD) at the platform layer. The goal is to ensure that all data pipelines can benefit from these mechanisms without duplicating effort.

First, let’s clarify what we are solving for: late-arriving and out-of-order data can disrupt real-time analytics and can lead to incorrect insights if not handled properly. Watermarks are a common mechanism used in stream processing frameworks like Apache Flink or Spark Streaming to track the latest known timestamp of received records. Restatement involves reprocessing old data as new information arrives, which is more resource-intensive but ensures correctness. SCD is typically employed for historical data in relational databases and can be extended to handle streaming data.

#### Assumptions

We are working with a large-scale data platform.
There is a mix of real-time and batch processing needs.
The goal is to maintain consistency across all pipelines, reduce redundancy, and ensure fault tolerance.

#### First Principles Solution
1. Watermarks

Watermarks can be implemented at the platform layer using stream processing frameworks that support this feature. By setting up watermarks, we can ensure that late data is properly handled without disrupting ongoing processing. The key steps are:

Define a watermark strategy in your streaming framework.
Ensure all pipelines read from streams with these watermarks enabled.
Configure the framework to drop or process late records based on the watermark.
2. Restatement Mechanisms

For situations where data needs to be reprocessed due to changes, a restatement mechanism can be employed. This involves:

Maintaining a versioned history of data at the platform level.
Configuring pipelines to query this history and reprocess old records as needed.
Using incremental or full reprocessing strategies based on the nature of the change.
3. Slowly Changing Dimensions (SCD)

Implementing SCD at the platform layer can handle out-of-order and late data effectively in a relational database environment. The steps are:

Define different types of SCDs (Type I, Type II, etc.) for handling changes.
Implement these types as database procedures or functions that pipelines can call.
Ensure all critical joins and aggregations account for the correct version of the dimension.
4. Platform-Level Policies

Define policies at the platform level to govern how data should be handled in terms of late arrival and out-of-order processing. This includes:

Defining watermark levels and thresholds.
Specifying conditions under which restatements are necessary.
Configuring SCD types for different types of changes.
5. Monitoring and Alerting

Set up monitoring and alerting at the platform level to track the health and consistency of data pipelines. This ensures that any issues with late or out-of-order data can be detected early and addressed promptly.

#### Closing Summary

By implementing a platform strategy involving watermarks, restatement mechanisms, SCDs, and robust policies, we can ensure consistent handling of late-arriving and out-of-order data across all pipelines without duplicating effort. This approach enhances the reliability and accuracy of our data processing infrastructure.

#### Common Mistakes

Failing to define clear policies and procedures for watermarking.
Overlooking the need for restatements in critical applications where historical correctness is paramount.
Not integrating SCDs effectively, leading to incorrect joins or aggregations.

#### Follow-Ups

Discuss specific use cases, trade-offs between different mechanisms, and potential performance optimizations. Also, explore how these strategies can be integrated with existing data governance frameworks.

**Comparison:**

```
Data Pipeline	Late Record Handling Strategy
Real-time	Watermark
Batch	Restatement
Data Type	SCD Type
Customer	Type II
Product	Type I
Sales	Type II, Type III
```

**Code:**

```python
# Example watermark implementation in Apache Flink
def on_timer(timestamp: int):
    # Process late records based on the current watermark
    pass

env = StreamExecutionEnvironment.get_execution_environment()
watermark_strategy = EventTimeWatermarkStrategy.from_bounded_out_of orderness(Duration.ofSeconds(10))
```

**Example input / output:**

```text
Input:
- Stream of data records with timestamps.
- Watermarks indicating the latest known timestamp.

Output:
- Processed stream where late records are handled appropriately based on watermarks.
- Log entries for watermark updates and dropped/processed records.
```

**Data:**

```json
{
  "watermarks": {
    "stream1": {"threshold": "2023-10-01T12:00:00Z", "last_processed": "2023-10-01T15:00:00Z"},
    "stream2": {"threshold": "2023-10-02T10:00:00Z", "last_processed": "2023-10-02T14:00:00Z"}
  }
}
```

**Flow:**

1. No
2. Yes
3. Data Ingest
4. Is Data Late?
5. Process Data
6. Apply Watermarks
7. Reprocess if Necessary
8. Store in Historical DB

