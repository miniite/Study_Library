---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd build a Python incident-timeline generator that assembles the ordered sequence of alerts, deploys, and remediations for a postmortem, pulling from multiple data sources.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incident Enrichment Scripts](<../../../../TOOLS/Python/Python_Language/Incident_Enrichment_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a Python incident-timeline generator for an AIOps environment, we need to construct a system that can aggregate events like alerts, deploys, and remediations from various data sources into a chronological sequence suitable for postmortems. The key steps involve defining the structure of the timeline, integrating with diverse data sources, processing and filtering events, and finally, generating the timeline in a readable format.

#### Triage
What the Question Is Asking

The primary goal is to create a Python tool that can compile all relevant events from different systems into an incident timeline. The system should be able to fetch data from multiple sources (e.g., monitoring tools, deployment logs, ticketing systems), process it according to timestamps, and present a coherent sequence of events.

#### Assumptions
Data Sources: We assume there are multiple external APIs or log files that provide event data.
Event Structure: Events will have common fields like timestamp, event_type, and description.
Output Format: The timeline should be in a human-readable format, possibly JSON for easy parsing.

#### Solution
Define the Timeline Structure
Create a class or function to represent an event with attributes: timestamp, type, description.
Integrate Data Sources
Use Python’s requests library to fetch data from APIs.
Process log files using built-in file handling methods.
Process and Filter Events
Parse the fetched data into a common format (e.g., JSON).
Sort events by their timestamps.
Generate the Timeline
Use Pandas or similar libraries to organize and manipulate event data.
Format the timeline output in a readable manner.

#### First-Principles Solution
Define Event Class:
PYTHON
class Event:
    def __init__(self, timestamp, type, description):
        self.timestamp = timestamp
        self.type = type
        self.description = description

    def to_dict(self):
        return {
            "timestamp": self.timestamp,
            "type": self.type,
            "description": self.description
        }
Fetch and Parse Data:
PYTHON
import requests
from datetime import datetime

def fetch_data(url):
    response = requests.get(url)
    if response.status_code == 200:
        return [Event(timestamp, type, description) for timestamp, type, description in response.json()]
    else:
        raise Exception("Failed to fetch data from API")

def process_log_file(file_path):
    events = []
    with open(file_path, 'r') as file:
        # Simple log parsing
        for line in file:
            parts = line.split(',')
            if len(parts) == 3:  # Assuming a simple CSV format
                timestamp, type, description = map(str.strip, parts)
                try:
                    events.append(Event(datetime.strptime(timestamp, '%Y-%m-%d %H:%M:%S'), type, description))
                except ValueError:
                    print(f"Invalid timestamp in line: {line}")
    return events
Sort and Organize Events:
PYTHON
import pandas as pd

def combine_events(*events_lists):
    all_events = []
    for event_list in events_lists:
        all_events.extend(event_list)
    df = pd.DataFrame([event.to_dict() for event in all_events])
    sorted_df = df.sort_values(by='timestamp')
    return [Event(**row) for _, row in sorted_df.iterrows()]
Generate the Timeline:
PYTHON
def generate_timeline(events):
    timeline = []
    previous_time = None
    for event in events:
        if not previous_time or event.timestamp > previous_time:
            timeline.append(event)
            previous_time = event.timestamp
    return timeline

#### Closing Summary
+ Common Mistakes/Follow-Ups

By combining data from multiple sources and sorting events by their timestamps, we can create a comprehensive incident timeline. The key challenges include handling different formats of input data and ensuring the system is robust against errors in timestamp parsing.

#### Common mistakes
might include:

Overlooking edge cases where event timestamps are not properly formatted.
Not handling failures gracefully when fetching data from external sources.
Ignoring the possibility that events might have overlapping or conflicting information.

#### Follow-ups
could involve discussing how to handle real-time streaming of events, integrating with a database for persistent storage, and optimizing performance for large volumes of events.

**Comparison:**

| Timestamp          | Type  | Description                            |
|--------------------|-------|----------------------------------------|
| 2023-10-01 15:34:47| alert | Critical error detected                |
| 2023-10-01 16:00:05| deploy| New version deployed successfully      |

**Code:**

```python
class Event:
    def __init__(self, timestamp, type, description):
        self.timestamp = timestamp
        self.type = type
        self.description = description

    def to_dict(self):
        return {
            "timestamp": self.timestamp,
            "type": self.type,
            "description": self.description
        }

def fetch_data(url):
    response = requests.get(url)
    if response.status_code == 200:
        return [Event(timestamp, type, description) for timestamp, type, description in response.json()]
    else:
        raise Exception("Failed to fetch data from API")

def process_log_file(file_path):
    events = []
    with open(file_path, 'r') as file:
        # Simple log parsing
        for line in file:
            parts = line.split(',')
            if len(parts) == 3:  # Assuming a simple CSV format
                timestamp, type, description = map(str.strip, parts)
                try:
                    events.append(Event(datetime.strptime(timestamp, '%Y-%m-%d %H:%M:%S'), type, description))
                except ValueError:
                    print(f"Invalid timestamp in line: {line}")
    return events

def combine_events(*events_lists):
    all_events = []
    for event_list in events_lists:
        all_events.extend(event_list)
    df = pd.DataFrame([event.to_dict() for event in all_events])
    sorted_df = df.sort_values(by='timestamp')
    return [Event(**row) for _, row in sorted_df.iterrows()]

def generate_timeline(events):
    timeline = []
    previous_time = None
    for event in events:
        if not previous_time or event.timestamp > previous_time:
            timeline.append(event)
            previous_time = event.timestamp
    return timeline
```

**Example input / output:**

```text
# Sample input data
api_events = fetch_data("https://example.com/api/events")
log_events = process_log_file("path/to/logfile.log")

combined_events = combine_events(api_events, log_events)
timeline = generate_timeline(combined_events)

for event in timeline:
    print(f"{event.timestamp} - {event.type}: {event.description}")
```

**Data:**

```json
[
    {"timestamp": "2023-10-01 15:34:47", "type": "alert", "description": "Critical error detected"},
    {"timestamp": "2023-10-01 16:00:05", "type": "deploy", "description": "New version deployed successfully"}
]
```

**Flow:**

1. Yes
2. No
3. Fetch Data from API
4. Is JSON Valid?
5. Parse and Create Event Objects
6. Log Error, Skip Line
7. Combine with Log Events
8. Sort by Timestamp
9. Generate Timeline
10. Print Timeline Events

