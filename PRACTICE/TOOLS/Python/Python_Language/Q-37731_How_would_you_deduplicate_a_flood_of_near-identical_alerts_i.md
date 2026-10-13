---
type: "practice"
difficulty: "Intermediate"
---

# How would you deduplicate a flood of near-identical alerts in Python using a stable fingerprint, and what fields belong in the fingerprint versus what would cause false merges?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incident Enrichment Scripts](<../../../../TOOLS/Python/Python_Language/Incident_Enrichment_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To deduplicate a flood of near-identical alerts in Python, we need to create a stable fingerprint that uniquely identifies each alert while ignoring minor differences. This involves understanding which fields are critical and should be part of the fingerprint, versus those that can vary slightly without causing false merges.

#### Triage
Question Breakdown

The core task is to implement an efficient deduplication mechanism in Python for alerts. We need to decide on a stable identifier (fingerprint) and ensure it accurately captures the essential information while ignoring noise or minor differences. The key is to balance precision with performance, as creating fingerprints can be computationally expensive.

#### Assumptions
Alert Structure: Assume each alert has several fields such as timestamp, severity level, source system ID, message content, etc.
Fingerprint Generation: We will use a hash function (like SHA-256) to generate the fingerprint based on selected fields.
Performance: The solution should be efficient in terms of time and memory.
Step-by-Step Solution

Identify Critical Fields:

Timestamps: Ignore exact timestamps, as they can differ by milliseconds or seconds due to system clock skew. Use the date part only.
Severity Level: This is a critical field that should be included in the fingerprint.
Source System ID: Essential for distinguishing alerts from different sources.
Message Content: Include key parts of the message content, ignoring minor variations like trailing whitespace or small typos.

Generate Fingerprint:

Use hashlib to generate a SHA-256 hash based on the selected fields.
Combine the selected fields into a string and hash it.

#### Implementation
in Python:

PYTHON
import hashlib

def generate_fingerprint(alert):
    # Extract critical fields
    timestamp = alert['timestamp'].date()
    severity_level = alert['severity']
    source_system_id = alert['source_system_id']
    message_content = alert['message'].replace(' ', '')  # Remove spaces to reduce noise

    # Create a string combining the critical fields
    fingerprint_str = f"{timestamp},{severity_level},{source_system_id},{message_content}"

    # Generate SHA-256 hash
    fingerprint = hashlib.sha256(fingerprint_str.encode()).hexdigest()

    return fingerprint

# Example alert data
alerts = [
    {'timestamp': '2023-10-04 12:00:01', 'severity': 'CRITICAL', 'source_system_id': 'sys1', 'message': 'Disk space warning'},
    {'timestamp': '2023-10-04 12:00:05', 'severity': 'WARNING', 'source_system_id': 'sys1', 'message': 'Disk space warning.'},
    {'timestamp': '2023-10-04 12:01:00', 'severity': 'CRITICAL', 'source_system_id': 'sys2', 'message': 'CPU usage high'},
]

# Deduplicate alerts
fingerprint_map = {}
for alert in alerts:
    fingerprint = generate_fingerprint(alert)
    if fingerprint not in fingerprint_map:
        fingerprint_map[fingerprint] = [alert]
    else:
        fingerprint_map[fingerprint].append(alert)

# Print deduplicated results
for fingerprints, group_alerts in fingerprint_map.items():
    if len(group_alerts) > 1:
        print(f"Fingerprint: {fingerprints}, Alerts: {group_alerts}")

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution ensures that alerts are accurately grouped based on their core characteristics while ignoring minor variations. However, there are potential pitfalls:

Edge Cases: Handle edge cases where the message content is identical but the severity or source system ID differs.
Performance: Ensure that fingerprint generation and comparison are efficient enough to handle a flood of alerts in real-time scenarios.
False Merges: Be cautious with fields like timestamps, as ignoring them entirely can lead to false merges if there's significant time difference.

#### Common Mistakes
Including all fields without considering their impact on deduplication accuracy.
Overlooking edge cases that could cause false merges.
Ignoring performance optimizations for large-scale alerts.

**Comparison:**

| Alert ID | Severity Level | Source System ID | Message Content          |
|----------|---------------|-----------------|-------------------------|
| A1       | CRITICAL      | sys1            | Disk space warning      |
| B2       | WARNING       | sys1            | Disk space warning.     |
| C3       | CRITICAL      | sys2            | CPU usage high          |

**Code:**

```python
import hashlib

def generate_fingerprint(alert):
    # Extract critical fields
    timestamp = alert['timestamp'].date()
    severity_level = alert['severity']
    source_system_id = alert['source_system_id']
    message_content = alert['message'].replace(' ', '')  # Remove spaces to reduce noise

    # Create a string combining the critical fields
    fingerprint_str = f"{timestamp},{severity_level},{source_system_id},{message_content}"

    # Generate SHA-256 hash
    fingerprint = hashlib.sha256(fingerprint_str.encode()).hexdigest()

    return fingerprint
```

**Example input / output:**

```text
fingerprint_map = {}
for alert in alerts:
    fingerprint = generate_fingerprint(alert)
    if fingerprint not in fingerprint_map:
        fingerprint_map[fingerprint] = [alert]
    else:
        fingerprint_map[fingerprint].append(alert)

# Print deduplicated results
for fingerprints, group_alerts in fingerprint_map.items():
    if len(group_alerts) > 1:
        print(f"Fingerprint: {fingerprints}, Alerts: {group_alerts}")
```

**Data:**

```python
alerts = [
    {'timestamp': '2023-10-04 12:00:01', 'severity': 'CRITICAL', 'source_system_id': 'sys1', 'message': 'Disk space warning'},
    {'timestamp': '2023-10-04 12:00:05', 'severity': 'WARNING', 'source_system_id': 'sys1', 'message': 'Disk space warning.'},
    {'timestamp': '2023-10-04 12:01:00', 'severity': 'CRITICAL', 'source_system_id': 'sys2', 'message': 'CPU usage high'},
]
```

**Flow:**

1. Match - D[Mark asDuplicate];D
2. Input Alert
3. Generate Fingerprint
4. Compare Fingerprints
5. Handle False Merges

