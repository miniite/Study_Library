---
type: "practice"
difficulty: "Deep"
---

# How would you retrofit closed-loop automation onto a legacy monolith with no clean APIs where the only 'action' is a runbook someone runs by hand?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Safe Closed-loop Remediation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Safe_Closed-loop_Remediation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When retrofitting closed-loop automation onto a legacy monolith without clean APIs, the goal is to maintain system stability and gradually introduce modern practices while respecting existing constraints. The architecture must balance complexity with effectiveness, ensuring that automated processes can identify issues, take corrective actions, and revert if necessary, all while integrating seamlessly into the manual workflow.

The key challenge here is that we lack clear entry points or APIs into the monolith, making it difficult to programmatically interact with its internal state or execution flow. To address this, we need a layered approach that combines data extraction from logs and event streams, heuristic-based anomaly detection, and gradual automation of specific actions via external triggers.

First, let's outline the key components required for this retrofit:

Data Ingestion Layer: Extract relevant data from log files or event streams.
Anomaly Detection Engine: Implement heuristics to detect anomalies in system behavior.
Automated Response Mechanism: Develop scripts or workflows that can be triggered based on detected anomalies.
Manual Workaround Integration: Ensure these automated responses can coexist with existing manual runbooks, allowing for fallbacks.

To start, we would need to:

Assess the System State: Understand what aspects of the monolith's operation are most critical and where automation could be beneficial.
Identify Monitoring Points: Determine which logs or events might indicate an issue that requires intervention. This involves parsing log files to understand their structure and content.
Design Heuristic Rules: Develop rules to detect anomalies based on historical data or predefined thresholds.
Implement Automated Actions: Write scripts or workflows that can be triggered by these heuristic-based decisions.

Next, we'll break down the steps in more detail:

Data Ingestion:

Use tools like logstash or custom parsers to extract relevant logs and events from the monolith’s output.
For example, if the application writes errors to a log file, we can use a regular expression to capture these error messages.

Anomaly Detection:

Implement a heuristic model that monitors key metrics (e.g., response times, error rates) against baseline values.
Use machine learning techniques like anomaly detection algorithms to flag unusual patterns.

Automated Actions:

Develop scripts or workflows that can be automatically executed when an anomaly is detected.
These could range from simple restart commands for services to more complex state changes in the monolith.

Manual Workaround Integration:

Introduce a mechanism where manual interventions are logged and can override automated actions if necessary.
For example, a flag in the log file indicating that an action should be manually resolved before automatic correction is attempted.

Here’s a simplified example to illustrate:

Log Extraction
Anomaly Detection
Automated Action
Manual Override

By following this structured approach, we can incrementally introduce automation into the monolith without disrupting existing operations. The critical aspect is to ensure that each step is thoroughly tested and validated before deployment.

**Comparison:**

```
Step	Description
1	Log Extraction and Parsing
2	Anomaly Detection Logic Setup
3	Automated Action Scripting
4	Integration with Manual Workflows
```

**Code:**

```python
# Example script for log extraction using Python's logging library

import re
import json

def parse_logs(log_file):
    error_pattern = r"ERROR.*"
    with open(log_file, 'r') as file:
        logs = file.readlines()
        errors = [log for log in logs if re.match(error_pattern, log)]
    return errors

# Example anomaly detection heuristic
def detect_anomaly(errors):
    threshold = 5  # Number of errors considered anomalous
    if len(errors) > threshold:
        return True
    return False
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "log_file": "/var/log/app.log",
  "errors": [
    "ERROR: Service failed at 2023-10-01T14:00Z",
    "ERROR: Database connection timeout at 2023-10-01T15:00Z"
  ]
}

Output:

JSON
Copy
{
  "anomaly_detected": true,
  "actions_needed": [
    {"service_restart": "/path/to/service/restart"},
    {"database_check": "/path/to/database/check"}
  ]
}
```

**Data:**

```json
{
  "log_file": "/var/log/app.log",
  "errors": [
    "ERROR: Service failed at 2023-10-01T14:00Z",
    "ERROR: Database connection timeout at 2023-10-01T15:00Z"
  ]
}
```

**Flow:**

1. Log Ingestion
2. Parse Logs
3. Detect Anomalies
4. Trigger Actions
5. Manual Override
6. Revert if Necessary

