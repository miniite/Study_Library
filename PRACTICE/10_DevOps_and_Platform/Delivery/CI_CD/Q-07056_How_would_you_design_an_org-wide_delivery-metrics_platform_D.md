---
type: "practice"
difficulty: "Expert"
---

# How would you design an org-wide delivery-metrics platform (DORA plus your own) that teams trust, can't game, and that leadership uses for investment decisions rather than as a stick?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ DORA Metrics](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/DORA_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an org-wide delivery-metrics platform that teams trust and leadership utilizes for informed decision-making, we need to focus on robustness, transparency, and integrity of metrics. The platform will integrate DORA (Deployment Frequency, Lead Time, Change Failure Rate, Mean Time To Restore) metrics with additional custom ones that ensure a holistic view of software delivery efficiency. We'll leverage DevOps principles and automation tools to minimize gaming behaviors while ensuring the data is actionable.

The core idea here is to build a system where:

Metrics are consistently collected in real-time.
The platform ensures traceability and transparency, making it difficult for teams to manipulate or game metrics.
Leadership uses this information to make informed decisions on investment without resorting to punitive measures.

**Comparison:**

```
Team	Deployment Frequency	Lead Time (hours)	Change Failure Rate	Mean Time To Restore (minutes)
DevOps	5	240	10%	60
```

**Code:**

```python
# Example of a simple metric collector function using Flask
from flask import Flask, request
import json

app = Flask(__name__)

@app.route('/metrics', methods=['POST'])
def collect_metrics():
    payload = request.json
    # Validate and sanitize input to prevent gaming
    if not validate_metrics(payload):
        return "Invalid metrics", 400

    save_to_db(payload)
    return "Metrics received successfully", 201

def validate_metrics(metrics):
    # Simple validation logic (more complex in reality)
    required_keys = ['team', 'deployment_frequency', 'lead_time', 'change_failure_rate', 'mean_time_to_restore']
    if all(key in metrics for key in required_keys):
        return True
    return False

def save_to_db(payload):
    # Mock database save function
    with open('metrics.json', 'a') as f:
        json.dump(payload, f)
        f.write('\n')

if __name__ == '__main__':
    app.run(debug=True)
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "team": "DevOps",
  "deployment_frequency": 5,
  "lead_time": 240,
  "change_failure_rate": 0.1,
  "mean_time_to_restore": 60
}

Output:

CODE
Copy
Metrics received successfully
```

**Data:**

```json
{
  "team": "DevOps",
  "deployment_frequency": 5,
  "lead_time": 240,
  "change_failure_rate": 0.1,
  "mean_time_to_restore": 60
}
```

**Flow:**

1. Start
2. Validate Metrics
3. Save to DB
4. Reject
5. Log Received Metric
6. Return Error
7. End

