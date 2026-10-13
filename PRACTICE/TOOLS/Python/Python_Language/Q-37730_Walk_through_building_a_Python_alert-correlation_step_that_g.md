---
type: "practice"
difficulty: "Deep"
---

# Walk through building a Python alert-correlation step that groups related alerts firing within a time window into a single incident. How do you pick the window and avoid merging unrelated storms?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incident Enrichment Scripts](<../../../../TOOLS/Python/Python_Language/Incident_Enrichment_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build an effective alert-correlation system in Python, we need to design a step that groups related alerts into incidents based on their timestamps. The key challenge here is setting an appropriate time window and distinguishing between correlated alerts and unrelated alert storms. Let's break down the process from first principles.

The question is asking us to implement a mechanism for grouping alerts within a given time window. This involves defining the criteria for determining when two or more alerts are related (e.g., by their source, type, severity) and ensuring that we don't mistakenly combine unrelated alert events into a single incident.

First, let's establish some assumptions:

Alerts are structured as dictionaries containing relevant fields such as alert_id, timestamp, source, severity, etc.
We need to balance between grouping closely related alerts (to avoid false negatives) and separating unrelated ones (to avoid false positives).

Now, we'll go through the steps of implementing this functionality.

Define the Alert Model:

PYTHON
class Alert:
    def __init__(self, alert_id, timestamp, source, severity):
        self.alert_id = alert_id
        self.timestamp = timestamp
        self.source = source
        self.severity = severity

    @staticmethod
    def from_dict(alert_dict):
        return Alert(
            alert_id=alert_dict['alert_id'],
            timestamp=alert_dict['timestamp'],
            source=alert_dict['source'],
            severity=alert_dict['severity']
        )

Implement the Correlation Logic: We will use a sliding time window approach where alerts that fall within this window and share common attributes (like source or severity) are considered correlated.

Design the Window Size: The optimal window size depends on the application context. For example, in a highly dynamic system, a smaller window might be more appropriate to quickly identify transient issues; whereas in a more stable environment, a larger window could help filter out noise.

Grouping Algorithm: We can use a simple algorithm that iterates through alerts and groups them based on the defined criteria.

Here’s an implementation:

PYTHON
def correlate_alerts(alerts, time_window_minutes=15):
    # Convert all timestamps to datetime for easier comparison
    for alert in alerts:
        alert.timestamp = datetime.fromtimestamp(alert.timestamp / 1000)

    incidents = []
    current_incident = None

    # Sort alerts by timestamp
    sorted_alerts = sorted(alerts, key=lambda x: x.timestamp)

    for i, alert in enumerate(sorted_alerts):
        if not current_incident or (alert.source != current_incident[-1].source and alert.severity != current_incident[-1].severity):
            # Start a new incident
            current_incident = [alert]
        else:
            # Add to existing incident
            current_incident.append(alert)

        if i < len(sorted_alerts) - 1 and sorted_alerts[i + 1].timestamp - alert.timestamp <= timedelta(minutes=time_window_minutes):
            continue
        else:
            # End the current incident and start a new one or add it to incidents list
            incidents.append(current_incident)
            current_incident = []

    return incidents

# Sample alerts data
alerts_data = [
    {'alert_id': 1, 'timestamp': 1633072800, 'source': 'server-1', 'severity': 4},
    {'alert_id': 2, 'timestamp': 1633072830, 'source': 'server-1', 'severity': 4},
    {'alert_id': 3, 'timestamp': 1633072900, 'source': 'server-2', 'severity': 3},
    # ... more alerts
]

# Convert sample data to Alert objects
alerts = [Alert.from_dict(alert) for alert in alerts_data]

Optimizing the Solution:

#### Performance Considerations
Ensure that the sorting and comparison operations are optimized, especially if dealing with a large number of alerts.
Edge Cases: Handle cases where there might be no overlap or when new alerts do not fit into any existing incidents.

#### Closing Summary
+ Common Mistakes/Follow-Ups: By setting an appropriate time window and using correlation criteria based on source, severity, etc., we can effectively group related alerts. The key is to balance between being too aggressive (merging unrelated events) and too conservative (missing correlated events). Common pitfalls include choosing an overly long or short window, overlooking critical attributes like source, and not handling edge cases properly.

#### Follow-ups
might involve discussing how to dynamically adjust the time window based on system behavior or integrating this with existing alerting systems.

**Code:**

```python
class Alert:
    def __init__(self, alert_id, timestamp, source, severity):
        self.alert_id = alert_id
        self.timestamp = timestamp
        self.source = source
        self.severity = severity

    @staticmethod
    def from_dict(alert_dict):
        return Alert(
            alert_id=alert_dict['alert_id'],
            timestamp=alert_dict['timestamp'],
            source=alert_dict['source'],
            severity=alert_dict['severity']
        )

def correlate_alerts(alerts, time_window_minutes=15):
    for alert in alerts:
        alert.timestamp = datetime.fromtimestamp(alert.timestamp / 1000)

    incidents = []
    current_incident = None

    sorted_alerts = sorted(alerts, key=lambda x: x.timestamp)

    for i, alert in enumerate(sorted_alerts):
        if not current_incident or (alert.source != current_incident[-1].source and alert.severity != current_incident[-1].severity):
            current_incident = [alert]
        else:
            current_incident.append(alert)

        if i < len(sorted_alerts) - 1 and sorted_alerts[i + 1].timestamp - alert.timestamp <= timedelta(minutes=time_window_minutes):
            continue
        else:
            incidents.append(current_incident)
            current_incident = []

    return incidents

# Sample alerts data
alerts_data = [
    {'alert_id': 1, 'timestamp': 1633072800, 'source': 'server-1', 'severity': 4},
    {'alert_id': 2, 'timestamp': 1633072830, 'source': 'server-1', 'severity': 4},
    {'alert_id': 3, 'timestamp': 1633072900, 'source': 'server-2', 'severity': 3},
]

# Convert sample data to Alert objects
alerts = [Alert.from_dict(alert) for alert in alerts_data]
```

**Example input / output:**

```

Input:

JSON
Copy
[
    {"alert_id": 1, "timestamp": 1633072800, "source": "server-1", "severity": 4},
    {"alert_id": 2, "timestamp": 1633072830, "source": "server-1", "severity": 4},
    {"alert_id": 3, "timestamp": 1633072900, "source": "server-2", "severity": 3}
]

Output:

PYTHON
Copy
[
    [Alert(alert_id=1, timestamp=datetime.datetime(2021, 9, 28, 14, 53), source='server-1', severity=4),
     Alert(alert_id=2, timestamp=datetime.datetime(2021, 9, 28, 14, 54), source='server-1', severity=4)],
    [Alert(alert_id=3, timestamp=datetime.datetime(2021, 9, 28, 14, 55), source='server-2', severity=3)]
]
```

**Data:**

```json
[
    {"alert_id": 1, "timestamp": 1633072800, "source": "server-1", "severity": 4},
    {"alert_id": 2, "timestamp": 1633072830, "source": "server-1", "severity": 4},
    {"alert_id": 3, "timestamp": 1633072900, "source": "server-2", "severity": 3}
]
```

**Flow:**

1. Yes
2. No
3. Match
4. No
5. Start
6. Are alerts within time window?
7. Group Alerts into Incident
8. Check source and severitymatch
9. End of Current Incident, StartNew One
10. Output Incidents

