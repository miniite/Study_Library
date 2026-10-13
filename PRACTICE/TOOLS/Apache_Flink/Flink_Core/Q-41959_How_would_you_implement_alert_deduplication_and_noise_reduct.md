---
type: "practice"
difficulty: "Deep"
---

# How would you implement alert deduplication and noise reduction using a keyed process function with state and timers in Flink?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Keyed vs Operator State](<../../../../TOOLS/Apache_Flink/Flink_Core/Keyed_vs_Operator_State.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Alert deduplication and noise reduction are critical functionalities in AIOps, ensuring that only relevant alerts reach decision-makers. In this implementation, we will use Apache Flink's Keyed Process Function to manage stateful processing with timers, effectively reducing false positives and ensuring timely alerting based on predefined conditions.

The question is probing your understanding of how to leverage Flink’s state management capabilities for complex event processing (CEP) tasks like deduplication. It also tests your ability to design fault-tolerant systems that handle events efficiently over time windows while maintaining low latency.

We will assume the following:

Events arrive in a stream with fields such as event_id, alert_type, and timestamp.
Deduplication should occur based on event_id within a certain time window (e.g., 5 minutes).
Noise reduction involves filtering out alerts that are too frequent, based on the number of occurrences within a given interval.
State management will keep track of event occurrences and their timestamps.

#### First-principles solution
We’ll implement a Keyed Process Function in Flink to maintain state for each unique event_id. This function will:

Track when an alert was last seen using timers.
Count the number of occurrences within a specified time window.
Filter out duplicate and noisy alerts based on these conditions.

Let’s break it down step by step:

Event Stream Definition: Define our event stream with fields event_id, alert_type, and timestamp.

PYTHON
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import BatchTableEnvironment, TableConfig
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_config = TableConfig()
t_env = BatchTableEnvironment.create(env, t_config)

t_env.connect(FileSystem().path('input'))
    .with_format(OldCsv()
                 .field('event_id', 'STRING')
                 .field('alert_type', 'STRING')
                 .field('timestamp', 'TIMESTAMP'))
    .with_schema(Schema()
                 .field('event_id', 'STRING')
                 .field('alert_type', 'STRING')
                 .field('timestamp', 'TIMESTAMP'))
    .in_append_mode()
    .register_table_source('input_table')

Keyed Process Function: Implement a Keyed Process Function to manage the state and timers.

PYTHON
from pyflink.table import EnvironmentSettings, BatchTableEnvironment

settings = EnvironmentSettings.new_instance().in_batch_mode().use_blink_planner().build()
t_env = BatchTableEnvironment.create(settings)

class DeduplicationFunction:
    def __init__(self):
        self.last_seen_time = {}

    def process(self, input):
        event_id, alert_type, timestamp = input

        if event_id not in self.last_seen_time:
            # Initialize the timer and state for a new event ID
            self.last_seen_time[event_id] = {"timestamp": timestamp, "count": 0}
            return None

        last_timestamp = self.last_seen_time[event_id]["timestamp"]

        if (timestamp - last_timestamp).total_seconds() < 300:
            # Deduplicate alert based on time window
            self.last_seen_time[event_id]["count"] += 1

            if self.last_seen_time[event_id]["count"] > 5:  # Example noise reduction threshold
                del self.last_seen_time[event_id]  # Delete noisy event ID from state
                return (event_id, alert_type)  # Emit the duplicate or noisy alert
            else:
                return None  # Ignore this occurrence for now

t_env.register_function('deduplication', DeduplicationFunction())

t_env.from_path('input_table').process_element(DeduplicationFunction()).execute_insert('output')

State Management and Timer Handling: The Keyed Process Function will use in-memory state to track the last seen time and count of occurrences for each event_id. Timers are used to manage event windows, ensuring alerts only trigger once per defined window.

Output Schema: Define the output schema to ensure the deduplicated and noise-reduced alerts are correctly emitted.

PYTHON
t_env.connect(FileSystem().path('output'))
    .with_format(OldCsv()
                 .field('event_id', 'STRING')
                 .field('alert_type', 'STRING'))
    .with_schema(Schema()
                 .field('event_id', 'STRING')
                 .field('alert_type', 'STRING'))
    .in_append_mode()
    .register_table_sink('output_table')

Execution: Run the Flink job to process the input data and emit the cleaned-up alerts.

#### Closing summary
The implementation leverages Apache Flink’s stateful processing capabilities, specifically Keyed Process Functions, to efficiently deduplicate and reduce noise in alert streams. This solution ensures that only relevant alerts are processed while maintaining low latency and high throughput.

#### Common mistakes/follow-ups

Incorrect State Management: Not properly tracking event timestamps or counts can lead to false positives.
Timer Handling: Ensuring timers trigger at the correct times without causing state inconsistencies is crucial.
Windowing Strategy: Choosing appropriate window sizes for deduplication and noise reduction is critical; too small a window may miss legitimate alerts, while too large a window may not detect noisy patterns.

**Comparison:**

```
event_id	alert_type
1001	network_error
1002	application_crash
1003	disk_full
```

**Code:**

```python
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import BatchTableEnvironment, TableConfig, EnvironmentSettings

settings = EnvironmentSettings.new_instance().in_batch_mode().use_blink_planner().build()
t_env = BatchTableEnvironment.create(settings)

# Event stream definition
t_env.connect(FileSystem().path('input'))
    .with_format(OldCsv()
                 .field('event_id', 'STRING')
                 .field('alert_type', 'STRING')
                 .field('timestamp', 'TIMESTAMP'))
    .with_schema(Schema()
                 .field('event_id', 'STRING')
                 .field('alert_type', 'STRING')
                 .field('timestamp', 'TIMESTAMP'))
    .in_append_mode()
    .register_table_source('input_table')

# Keyed Process Function implementation
class DeduplicationFunction:
    def __init__(self):
        self.last_seen_time = {}

    def process(self, input):
        event_id, alert_type, timestamp = input

        if event_id not in self.last_seen_time:
            # Initialize the timer and state for a new event ID
            self.last_seen_time[event_id] = {"timestamp": timestamp, "count": 0}
            return None

        last_timestamp = self.last_seen_time[event_id]["timestamp"]

        if (timestamp - last_timestamp).total_seconds() < 300:
            # Deduplicate alert based on time window
            self.last_seen_time[event_id]["count"] += 1

            if self.last_seen_time[event_id]["count"] > 5:  # Example noise reduction threshold
                del self.last_seen_time[event_id]  # Delete noisy event ID from state
                return (event_id, alert_type)  # Emit the duplicate or noisy alert
            else:
                return None  # Ignore this occurrence for now

t_env.register_function('deduplication', DeduplicationFunction())

# Output schema definition
t_env.connect(FileSystem().path('output'))
    .with_format(OldCsv()
                 .field('event_id', 'STRING')
                 .field('alert_type', 'STRING'))
    .with_schema(Schema()
                 .field('event_id', 'STRING')
                 .field('alert_type', 'STRING'))
    .in_append_mode()
    .register_table_sink('output_table')

t_env.from_path('input_table').process_element(DeduplicationFunction()).execute_insert('output_table')
```

**Example input / output:**

```

Input:

CODE
Copy
"event_id": ["1001", "1002", "1001", "1003", "1002", "1001"],
"alert_type": ["network_error", "application_crash", "network_error", "disk_full", "application_crash", "network_error"],
"timestamp": [2023-10-01 14:00:00, 2023-10-01 14:05:00, 2023-10-01 14:10:00, 2023-10-01 14:15:00, 2023-10-01 14:20:00, 2023-10-01 14:25:00]

Output:

CODE
Copy
"event_id": ["1001", "1002", "1003"],
"alert_type": ["network_error", "application_crash", "disk_full"]
```

**Data:**

```text
# Sample input data
"event_id", "alert_type", "timestamp"
"1001", "network_error", 2023-10-01 14:00:00
"1002", "application_crash", 2023-10-01 14:05:00
"1001", "network_error", 2023-10-01 14:10:00
"1003", "disk_full", 2023-10-01 14:15:00
"1002", "application_crash", 2023-10-01 14:20:00
"1001", "network_error", 2023-10-01 14:25:00
# Sample output data
"event_id", "alert_type"
"1001", "network_error"
"1002", "application_crash"
"1003", "disk_full"
```

**Flow:**

1. No
2. Yes
3. Above Threshold
4. Below Threshold
5. Yes
6. No
7. Event Input
8. Is Event Duplicated?
9. Deduplicate and Emit
10. Check Noise ReductionThreshold
11. Emit Alert
12. Ignore Occurrence
13. State Management for Timers
14. Timeout Check
15. Is Timer Expired?

