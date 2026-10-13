---
type: "practice"
difficulty: "Intermediate"
---

# Your anomaly pipeline must not page on-call during a planned maintenance window. How do you stream maintenance schedules in and suppress alerts, and what happens to anomalies that start just before the window ends?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Provisional and Final Results](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we need to design an anomaly detection pipeline that can handle planned maintenance windows by suppressing relevant alerts during those times. The key challenge is ensuring that anomalies that start just before or during a maintenance window are not paged out and handled appropriately.

#### Triaging the Question

The question asks how to stream maintenance schedules into our anomaly detection system and suppress alerts for the duration of these maintenance periods. Additionally, it highlights the need to manage any anomalies that begin just as a maintenance window is ending.

#### Assumptions

The maintenance schedule is provided in a structured format.
The anomaly detection pipeline processes streaming data in real-time.
We have a system for alerting and paging on-call engineers.
The maintenance windows are known in advance and can be scheduled.
Anomalies are detected based on deviations from historical baselines.

#### First-Principles Solution

To solve this problem, we'll need to integrate the maintenance schedule into our anomaly detection pipeline. This involves several steps:

Modeling Maintenance Schedule:

Represent the maintenance schedules as a structured input, such as a JSON file or an API call.

Suppression Logic:

Implement logic within the pipeline that identifies when the current time falls within a maintenance window and suppresses alerts.

Handling Edge Cases:

Ensure that anomalies starting just before or during a maintenance window are properly managed to avoid false negatives.
Step-by-Step Solution
1. Model Maintenance Schedule

We can represent the maintenance schedule as follows:

JSON
{
  "maintenance_windows": [
    {
      "start_time": "2023-10-05T18:00:00Z",
      "end_time": "2023-10-05T22:00:00Z"
    },
    // Additional windows
  ]
}
2. Implement Suppression Logic in Pipeline

Assume our anomaly detection pipeline uses a continuous stream of data points, such as system metrics or log entries. We'll add logic to suppress alerts during maintenance periods.

PYTHON
import datetime
from typing import List, Tuple

# Sample maintenance schedule
maintenance_schedules = [
    {"start_time": "2023-10-05T18:00:00Z", "end_time": "2023-10-05T22:00:00Z"},
]

def is_within_maintenance_window(current_time: datetime.datetime, maintenance_windows: List[dict]) -> bool:
    for window in maintenance_windows:
        start = datetime.datetime.fromisoformat(window["start_time"])
        end = datetime.datetime.fromisoformat(window["end_time"])
        if start <= current_time < end:
            return True
    return False

# Example data point
data_point: Tuple[datetime.datetime, float] = (datetime.datetime.now(), 10.2)

if is_within_maintenance_window(data_point[0], maintenance_schedules):
    print("Maintenance window active, suppressing alert.")
else:
    # Normal anomaly detection logic here
    pass
3. Handling Edge Cases

When an anomaly starts just before a maintenance window ends, we need to ensure it's detected and handled correctly. One approach is to set up a buffer period where anomalies are still flagged but not paged out.

PYTHON
def handle_anomaly(data_point: Tuple[datetime.datetime, float], maintenance_windows: List[dict]) -> bool:
    current_time = data_point[0]
    latest_window_end = None

    for window in reversed(maintenance_schedules):
        if is_within_maintenance_window(current_time, [window]):
            latest_window_end = datetime.datetime.fromisoformat(window["end_time"])
            break

    if latest_window_end and (latest_window_end - current_time).seconds < 600:  # Buffer period of 10 minutes
        print("Anomaly detected close to maintenance window end. Flagging for review.")
        return True
    else:
        print("Normal anomaly handling logic here")
        return False

# Example usage
if handle_anomaly(data_point, maintenance_schedules):
    # Additional logging or storage
    pass

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the solution involves integrating a structured maintenance schedule into our anomaly detection pipeline and implementing suppression logic during these periods. We also need to account for anomalies that start just before a maintenance window ends by setting up a buffer period.

#### Common Mistakes

Failing to consider edge cases where anomalies could still be detected near the end of a maintenance window.
Not accounting for time zone differences in scheduling and timestamps.
Overlooking the potential impact on alert accuracy if maintenance windows are too broad or infrequent.

#### Follow-Ups
How would you handle overlapping maintenance schedules?
What about historical data processing? Should we adjust baselines during maintenance periods?
Can you discuss how this solution scales with increasing numbers of maintenance schedules and anomaly detection queries?

**Comparison:**

```
Data Point	Time	Value
	18:30:00	10.2
Maintenance Window	Start Time	End Time
	2023-10-05T18:00:00Z	2023-10-05T22:00:00Z
Anomaly Handling	Condition	Action
Maintenance	In maintenance	Suppress alert
		Buffer period active
Normal	Not in maintenance	Normal anomaly check
```

**Code:**

```python
import datetime

# Maintenance schedule representation
maintenance_schedules = [
    {"start_time": "2023-10-05T18:00:00Z", "end_time": "2023-10-05T22:00:00Z"},
]

def is_within_maintenance_window(current_time: datetime.datetime, maintenance_windows: List[dict]) -> bool:
    for window in maintenance_windows:
        start = datetime.datetime.fromisoformat(window["start_time"])
        end = datetime.datetime.fromisoformat(window["end_time"])
        if start <= current_time < end:
            return True
    return False

# Example data point
data_point: Tuple[datetime.datetime, float] = (datetime.datetime.now(), 10.2)

if is_within_maintenance_window(data_point[0], maintenance_schedules):
    print("Maintenance window active, suppressing alert.")
else:
    # Normal anomaly detection logic here
    pass

def handle_anomaly(data_point: Tuple[datetime.datetime, float], maintenance_windows: List[dict]) -> bool:
    current_time = data_point[0]
    latest_window_end = None

    for window in reversed(maintenance_schedules):
        if is_within_maintenance_window(current_time, [window]):
            latest_window_end = datetime.datetime.fromisoformat(window["end_time"])
            break

    if latest_window_end and (latest_window_end - current_time).seconds < 600:  # Buffer period of 10 minutes
        print("Anomaly detected close to maintenance window end. Flagging for review.")
        return True
    else:
        print("Normal anomaly handling logic here")
        return False

# Example usage
if handle_anomaly(data_point, maintenance_schedules):
    # Additional logging or storage
    pass
```

**Example input / output:**

```text
Input:
data_point = (datetime.datetime.now(), 10.2)
maintenance_schedules = [{"start_time": "2023-10-05T18:00:00Z", "end_time": "2023-10-05T22:00:00Z"}]

Output:
Maintenance window active, suppressing alert.

Anomaly detected close to maintenance window end. Flagging for review.
```

**Data:**

```json
{
  "maintenance_windows": [
    {"start_time": "2023-10-05T18:00:00Z", "end_time": "2023-10-05T22:00:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is current time within amaintenance window?
7. Suppress alert
8. Anomaly detected close tomaintenance window end?
9. Flag for review
10. Normal anomaly check
11. End

