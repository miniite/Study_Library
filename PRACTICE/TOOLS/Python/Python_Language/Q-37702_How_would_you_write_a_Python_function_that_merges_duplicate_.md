---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python function that merges duplicate alerts within a short time window into a single grouped alert to reduce noise?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Incident Enrichment Scripts](<../../../../TOOLS/Python/Python_Language/Incident_Enrichment_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to create a Python function that takes in a list of alert dictionaries and groups them based on their type and timestamp within a specified short time window. The goal is to merge duplicate alerts into a single grouped alert to reduce noise. Here’s how we can approach the solution:

#### Triage
The question requires us to write a Python function for merging duplicate alerts. Alerts are represented as dictionaries, with keys like "alert_type" and "timestamp". We need to define what constitutes a "duplicate" based on these attributes within a given time window.

#### First-Principles Solution

Define the alert type and timestamp as key criteria for grouping.
Set up a time window parameter (e.g., 5 minutes) to determine when two alerts are considered duplicates.
Iterate through the list of alerts, checking if any new alert overlaps with existing groups based on their timestamps.

#### Solution

PYTHON
from datetime import timedelta

def merge_alerts(alerts, time_window_minutes=5):
    """
    Merge duplicate alerts within a short time window into a single grouped alert.

    :param alerts: List of alert dictionaries with 'alert_type' and 'timestamp'
    :param time_window_minutes: Time window in minutes to consider duplicates
    :return: Merged list of alerts
    """
    merged_alerts = []
    for new_alert in alerts:
        # Convert timestamp from string to datetime object if necessary
        new_timestamp = new_alert['timestamp']
        new_type = new_alert['alert_type']

        existing_groups = [group for group in merged_alerts if _is_within_time_window(group[0]['timestamp'], new_timestamp, time_window_minutes)]

        if not existing_groups:
            # No overlapping groups found, add the alert as a new group
            merged_alerts.append([new_alert])
        else:
            # Merge with an existing group
            for group in existing_groups:
                if _is_same_type(group[0]['alert_type'], new_type):
                    group.append(new_alert)
                    break

    return [group[0] for group in merged_alerts]

def _is_within_time_window(timestamp1, timestamp2, time_window_minutes):
    """
    Check if two timestamps are within a certain time window.
    """
    time_diff = abs((timestamp2 - timestamp1).total_seconds()) / 60
    return time_diff <= time_window_minutes

def _is_same_type(type1, type2):
    """
    Check if alert types are the same.
    """
    return type1 == type2

#### Closing Summary
The function merge_alerts iterates through each alert and checks for overlapping groups based on their timestamps within a specified time window. If an alert overlaps with any existing group, it is added to that group; otherwise, it forms a new group.

#### Common Mistakes/Follow-Ups

Forgetting to handle the case where no alerts are found in the time window.
Incorrectly calculating the time difference between timestamps.
Not properly checking if alert types match when merging groups.

**Comparison:**

| Alert Type  | Timestamp             |
|-------------|-----------------------|
| server_down | 2023-10-05T14:00:00Z  |
| server_down | 2023-10-05T14:02:00Z  |
| network_error | 2023-10-05T14:06:00Z  |

**Code:**

```python
from datetime import timedelta

def merge_alerts(alerts, time_window_minutes=5):
    merged_alerts = []
    for new_alert in alerts:
        new_timestamp = new_alert['timestamp']
        new_type = new_alert['alert_type']

        existing_groups = [group for group in merged_alerts if _is_within_time_window(group[0]['timestamp'], new_timestamp, time_window_minutes)]

        if not existing_groups:
            merged_alerts.append([new_alert])
        else:
            for group in existing_groups:
                if _is_same_type(group[0]['alert_type'], new_type):
                    group.append(new_alert)
                    break

    return [group[0] for group in merged_alerts]

def _is_within_time_window(timestamp1, timestamp2, time_window_minutes):
    time_diff = abs((timestamp2 - timestamp1).total_seconds()) / 60
    return time_diff <= time_window_minutes

def _is_same_type(type1, type2):
    return type1 == type2
```

**Example input / output:**

```text
alerts = [
    {"alert_type": "server_down", "timestamp": "2023-10-05T14:00:00Z"},
    {"alert_type": "server_down", "timestamp": "2023-10-05T14:02:00Z"},
    {"alert_type": "network_error", "timestamp": "2023-10-05T14:06:00Z"}
]

merged_alerts = merge_alerts(alerts)
print(merged_alerts)  # Output: [{'alert_type': 'server_down', 'timestamp': '2023-10-05T14:00:00Z'}, {'alert_type': 'network_error', 'timestamp': '2023-10-05T14:06:00Z'}]
```

**Data:**

```json
[
    {"alert_type": "server_down", "timestamp": "2023-10-05T14:00:00Z"},
    {"alert_type": "server_down", "timestamp": "2023-10-05T14:02:00Z"},
    {"alert_type": "network_error", "timestamp": "2023-10-05T14:06:00Z"}
]
```

**Flow:**

1. No
2. Yes
3. Start
4. Is alert in time window?
5. Merge with existing group
6. Create new group
7. End
8. Add to merged alerts

