---
type: "practice"
difficulty: "Deep"
---

# A streaming pipeline processes each event correctly but downstream aggregates drift over time. How do you detect and correct slow accumulating error?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming Correctness and Drift](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Streaming_Correctness_and_Drift.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of slow, accumulating errors in a streaming pipeline, we need to understand that batch processing inherently lags behind real-time updates, whereas streams provide more immediate feedback but can introduce complexity due to out-of-order events and state management. The core challenge here is detecting the drift in downstream aggregates and correcting it without disrupting ongoing operations.

#### Triage

The question is probing our understanding of how data inconsistencies manifest in streaming pipelines and how we can identify and mitigate these issues through robust monitoring and correction mechanisms. Key assumptions include:

The system already handles events correctly on a per-event basis.
Aggregates are calculated over time windows or sliding windows, which can lead to drift if not managed properly.

#### Solution

First-principles thinking dictates that the solution involves three main components: Detection, Correction, and Prevention.

Detection: We need a reliable way to detect when aggregate values have drifted from their expected state.
Correction: Once detected, we must correct the error efficiently without causing additional issues in the streaming pipeline.
Prevention: Implementing strategies to prevent such errors from occurring again is crucial for long-term reliability.
Step-by-Step Solution
1. Detection

Detection involves comparing current aggregate values with expected or baseline values. Here are a few methods:

Sliding Window Aggregation: Compare the current window's aggregation against previous windows.
Baseline Comparison: Maintain a snapshot of known good states and periodically compare current states to these baselines.
Anomaly Detection: Use statistical models like moving averages, standard deviations, or machine learning techniques (e.g., Isolation Forests) to flag outliers.
2. Correction

Correction requires addressing the detected drift in an efficient manner:

Incremental Adjustments: Update aggregates incrementally rather than recalculating from scratch.
Backfilling: If possible, backfill past data to correct historical aggregations.
Reprocessing: Re-run recent events through the pipeline with a corrected state.
3. Prevention

Prevention strategies include:

Event Timestamps and Ordering: Ensure proper handling of event timestamps and ordering to avoid out-of-order issues.
Watermarking: Implement watermarking techniques to handle late arriving data gracefully.
State Management: Use reliable state management systems like Apache Kafka's KTable or Apache Flink's checkpointing.

#### Implementation
Example

Let's consider a simple example using Python with Apache Flink:

PYTHON
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

# Define schema for input and output
t_env.connect(FileSystem().path('input_path'))
     .with_format(OldCsv()
                  .field('value', DataTypes.STRING()))
     .with_schema(Schema()
                  .field('timestamp', DataTypes.TIMESTAMP())
                  .field('value', DataTypes.DOUBLE()))
     .register_table_source('input')

# Register table sink for output
t_env.connect(FileSystem().path('output_path'))
     .with_format(OldCsv()
                  .field('key', DataTypes.STRING())
                  .field('aggregated_value', DataTypes.DOUBLE()))
     .with_schema(Schema()
                  .field('key', DataTypes.STRING())
                  .field('aggregated_value', DataTypes.DOUBLE()))
     .register_table_sink('output')

# Define window and aggregation
t_env.from_path('input') \
   .window(TumblingProcessingTimeWindows.of(Time.seconds(5))) \
   .group_by('key') \
   .select('key, SUM(value) as aggregated_value') \
   .insert_into('output')

t_env.execute("Streaming Aggregation")

#### Closing Summary

To detect and correct slow accumulating errors in a streaming pipeline:

Detection: Use sliding window aggregation, baseline comparison, or anomaly detection.
Correction: Apply incremental adjustments, backfilling, or reprocessing.
Prevention: Implement event timestamps, watermarking, and robust state management.

#### Common mistakes
include overlooking the stateful nature of aggregations or failing to handle late arriving data properly.

Common Follow-Ups
How do you ensure your detection mechanism is not triggered by legitimate variations in data?
What strategies would you use if you had limited computing resources for reprocessing historical events?

**Comparison:**

```
Input Data	Output Aggregation
timestamp	key
value	aggregated_value
Timestamp	Key	Value
2023-10-01T10:00:00Z	k1	1.0
2023-10-01T10:01:00Z	k1	3.0
Key	Aggregated Value
k1	3.0
Key	Aggregated Value
k1	3.0
```

**Code:**

```python
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

# Define schema for input and output
t_env.connect(FileSystem().path('input_path'))
     .with_format(OldCsv()
                  .field('value', DataTypes.STRING()))
     .with_schema(Schema()
                  .field('timestamp', DataTypes.TIMESTAMP())
                  .field('value', DataTypes.DOUBLE()))
     .register_table_source('input')

# Register table sink for output
t_env.connect(FileSystem().path('output_path'))
     .with_format(OldCsv()
                  .field('key', DataTypes.STRING())
                  .field('aggregated_value', DataTypes.DOUBLE()))
     .with_schema(Schema()
                  .field('key', DataTypes.STRING())
                  .field('aggregated_value', DataTypes.DOUBLE()))
     .register_table_sink('output')

# Define window and aggregation
t_env.from_path('input') \
   .window(TumblingProcessingTimeWindows.of(Time.seconds(5))) \
   .group_by('key') \
   .select('key, SUM(value) as aggregated_value') \
   .insert_into('output')

t_env.execute("Streaming Aggregation")
```

**Example input / output:**

```text
Input:
timestamp, value
2023-10-01T10:00:00Z, 1.0
2023-10-01T10:01:00Z, 2.0

Output (expected):
key, aggregated_value
k1, 3.0
```

**Data:**

```json
{
  "input_path": "path/to/input",
  "output_path": "path/to/output"
}
```

**Flow:**

1. Yes
2. No
3. Error Detected
4. No Error
5. Event Receival
6. Is Event Within CurrentWindow?
7. Update Aggregate
8. Save State for Later Use
9. Check Against Baseline orAnomaly Detection Model
10. Correct Error Incrementally
11. Output Aggregation
12. Reprocess Events if Necessary
13. Watermark Event to HandleLate Arrivals
14. Incremental Update of State
15. Continue Processing NewEvents

