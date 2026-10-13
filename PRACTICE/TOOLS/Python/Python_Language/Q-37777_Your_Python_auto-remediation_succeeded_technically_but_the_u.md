---
type: "practice"
difficulty: "Deep"
---

# Your Python auto-remediation succeeded technically but the underlying issue recurred an hour later. How would you detect recurrence and escalate to a human instead of blindly re-remediating forever?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Dead Letter and Replay Systems](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to create a system that detects when an auto-remediation issue recurs and escalates to a human analyst rather than continuing to automatically remediate indefinitely. The key steps involve detecting the recurrence of the issue, assessing its severity, and deciding whether manual intervention is necessary.

#### Triaging the Question

The question probes how we can dynamically adjust our response strategy based on system state changes. We need to define thresholds or conditions under which a recurring issue should be escalated from an automated process to human oversight. This involves:

Detection Logic: Identifying that the same error condition has reoccurred.
Severity Assessment: Determining if the recurrence is significant enough to warrant manual attention.
Escalation Mechanism: Creating a mechanism to notify and involve humans in the decision-making process.

#### Assumptions
The system already has some form of auto-remediation logic in place.
We have access to logs, monitoring data, or other relevant information about previous and current states.
There is a human analyst available during working hours who can be contacted through predefined channels (e.g., Slack, email).

#### First-Principles Solution

Detection of Recurrence:

Monitor the system state at regular intervals to detect changes or recurring issues.
Use historical data and thresholds to determine if a recurrence has occurred.

Severity Assessment:

Define severity levels based on past incidents (e.g., critical, high, medium, low).
Implement a scoring mechanism that considers factors like impact duration, frequency, and previous outcomes.

Escalation Mechanism:

If the recurrence meets certain criteria (higher severity score), trigger an escalation process.
This can involve sending alerts to specific individuals or teams via predefined communication channels.
Step-by-Step Solution

Log Analysis:

PYTHON
import pandas as pd

def load_logs(file_path):
    return pd.read_csv(file_path)

logs = load_logs('system_logs.csv')

Detect Recurrence:

Compare current log entries with previous ones to identify recurring issues.
PYTHON
from datetime import timedelta

def detect_recurrence(current_logs, historical_logs, threshold_hours=1):
    for index, row in current_logs.iterrows():
        if (pd.to_datetime(row['timestamp']) -
            pd.to_datetime(historical_logs.loc[historical_logs['issue'] == row['issue']]['timestamp'].max())).total_seconds() < 3600 * threshold_hours:
            return True
    return False

recurrence = detect_recurrence(current_logs, historical_logs)

Severity Assessment:

Assign a severity score based on the issue and its history.
PYTHON
def assign_severity(log_entry):
    if 'critical' in log_entry['issue']:
        return 5
    elif 'high' in log_entry['issue']:
        return 4
    elif 'medium' in log_entry['issue']:
        return 3
    else:
        return 1

severity_score = assign_severity(log_entry)

Escalation Mechanism:

Send an alert if the severity score exceeds a threshold.
PYTHON
def escalate_issue(severity_score, threshold=4):
    if severity_score >= threshold:
        send_alert(f'Severity: {severity_score}. Issue requires human intervention.')
    else:
        print('Issue is being automatically remediated.')

escalate_issue(severity_score)

#### Closing Summary
and Common Mistakes

This solution ensures that the system can adapt to recurring issues by detecting them, assessing their severity, and escalating only when necessary. Common mistakes include not defining clear thresholds for severity or failing to implement an effective escalation mechanism.

To enhance this further:

Implement a machine learning model to predict issue recurrence.
Use real-time monitoring tools to continuously update logs.
Establish a robust on-call schedule to ensure human analysts are always available.

**Comparison:**

```
Severity	Description
1	Low
3	Medium
4	High
5	Critical
Issue	Timestamp
critical	2023-10-15 13:59
high	2023-10-15 14:00
```

**Code:**

```python
import pandas as pd

def load_logs(file_path):
    return pd.read_csv(file_path)

def detect_recurrence(current_logs, historical_logs, threshold_hours=1):
    for index, row in current_logs.iterrows():
        if (pd.to_datetime(row['timestamp']) -
            pd.to_datetime(historical_logs.loc[historical_logs['issue'] == row['issue']]['timestamp'].max())).total_seconds() < 3600 * threshold_hours:
            return True
    return False

def assign_severity(log_entry):
    if 'critical' in log_entry['issue']:
        return 5
    elif 'high' in log_entry['issue']:
        return 4
    elif 'medium' in log_entry['issue']:
        return 3
    else:
        return 1

def escalate_issue(severity_score, threshold=4):
    if severity_score >= threshold:
        send_alert(f'Severity: {severity_score}. Issue requires human intervention.')
    else:
        print('Issue is being automatically remediated.')

logs = load_logs('system_logs.csv')
recurrence = detect_recurrence(logs, logs)
log_entry = {'issue': 'critical', 'timestamp': '2023-10-15 14:00'}
severity_score = assign_severity(log_entry)
escalate_issue(severity_score)
```

**Example input / output:**

```text
logs = load_logs('system_logs.csv')
recurrence = detect_recurrence(logs, logs)
log_entry = {'issue': 'critical', 'timestamp': '2023-10-15 14:00'}
severity_score = assign_severity(log_entry)
escalate_issue(severity_score)

# Output
Recurrence detected.
Severity: 5. Issue requires human intervention.
```

**Data:**

```json
{
    "system_logs.csv": [
        {"issue": "critical", "timestamp": "2023-10-15 13:59"},
        {"issue": "high", "timestamp": "2023-10-15 14:00"}
    ]
}
```

**Flow:**

1. Yes
2. High Score
3. No
4. Load Logs
5. Detect Recurrence?
6. Assign Severity
7. Escalate Issue
8. Continue Remediation

