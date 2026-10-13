---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up governance so that an AIOps model's decision to page a human at 3am vs. auto-handle is itself audited for fairness across on-call teams — preventing systematic overload of certain teams?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-2/3 · [↑ Human in the Loop Governance for AIOps](<../../../../11_AI_and_ML/Responsible_AI/AI_Governance/Human_in_the_Loop_Governance_for_AIOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

To ensure fair and responsible AI governance in an AIOps setting, we need a robust system that audits decisions made by models to page humans at 3am or auto-handle incidents. This involves implementing logging, monitoring, and fairness metrics across different on-call teams to prevent systematic biases.

#### Triaging the Question

The core of this question is about establishing governance mechanisms for AI-driven decision-making in AIOps, with a focus on ensuring fairness and preventing disproportionate burdening of certain on-call teams. The interviewer is probing your understanding of AI governance principles, fair machine learning practices, and practical implementation strategies.

#### Assumptions
We have an existing AIOps model that makes decisions to page humans or auto-handle incidents.
There are multiple on-call teams with varying responsibilities.
We need a system that can log, monitor, and analyze these decisions over time.
The goal is to detect and address any unfairness in the decision-making process.
Step-by-Step Solution:
1. Define Fairness Metrics

First, define what "fair" means for your organization. Common fairness metrics include:

Proportionality: Ensure that each on-call team has an equal probability of being paged.
Equity: Minimize the disparity in the number of incidents each team handles.
2. Implement Logging and Monitoring

Implement a logging system to capture all decisions made by the AIOps model, including:

PYTHON
import json

def log_decision(team_id, decision):
    with open('decision_logs.json', 'a') as log_file:
        entry = {'team_id': team_id, 'decision': decision, 'timestamp': datetime.now().isoformat()}
        json.dump(entry, log_file)
        log_file.write('\n')
3. Track Decision History

Store the logged decisions in a database or file for easy access and analysis.

SQL
CREATE TABLE DecisionLog (
    id SERIAL PRIMARY KEY,
    team_id INT NOT NULL,
    decision VARCHAR(255) NOT NULL,
    timestamp TIMESTAMP NOT NULL
);
4. Develop an Audit Tool

Create a tool to periodically analyze the logs for fairness metrics:

PYTHON
from datetime import datetime, timedelta

def audit_fairness(start_date=None, end_date=None):
    if start_date is None or end_date is None:
        now = datetime.now()
        start_date = now - timedelta(days=7)
        end_date = now

    log_file = open('decision_logs.json', 'r')
    total_paged = 0
    team_counts = {}

    for line in log_file:
        entry = json.loads(line.strip())
        if start_date <= datetime.fromisoformat(entry['timestamp']) < end_date and entry['decision'] == "page":
            team_id = entry['team_id']
            total_paged += 1
            if team_id not in team_counts:
                team_counts[team_id] = 0
            team_counts[team_id] += 1

    log_file.close()

    fairness_report = {
        'total_paged': total_paged,
        'team_frequencies': {team: count for team, count in team_counts.items()}
    }
    return fairness_report
5. Set Fairness Thresholds

Define thresholds that indicate when the decision-making process might be unfair:

PYTHON
def check_thresholds(report):
    threshold = total_paged * 0.8  # Example: 80% of incidents should be handled by any team

    for team_id, count in report['team_frequencies'].items():
        if count > threshold:
            print(f"Team {team_id} is handling more than 80% of paged incidents.")
6. Integrate with Model Training and Validation

Ensure that the fairness metrics are integrated into the model training and validation process to prevent unfairness from being baked into new models.

PYTHON
from sklearn.model_selection import train_test_split

def train_model_with_fairness(X, y):
    X_train, X_val, y_train, y_val = train_test_split(X, y, test_size=0.2)

    model.fit(X_train, y_train)
    val_predictions = model.predict(X_val)

    # Evaluate fairness on validation data
    audit_fairness()
7. Continuous Monitoring and Feedback

Set up a continuous monitoring pipeline to regularly check for unfair decisions:

PYTHON
def monitor_fairness_periodically():
    while True:
        report = audit_fairness()
        check_thresholds(report)
        time.sleep(3600)  # Check every hour

#### Closing Summary
and Common Mistakes/Follow-Ups:

In summary, ensuring fairness in AI-driven decision-making involves implementing robust logging, monitoring tools, and continuous auditing. By defining clear metrics, integrating these into the model lifecycle, and setting up periodic checks, we can prevent systematic biases and ensure fair treatment of all on-call teams.

#### Common Mistakes

Failing to Define Clear Fairness Metrics: Without a well-defined set of fairness criteria, it's hard to measure or address potential biases.
Overlooking Continuous Monitoring: One-time audits are insufficient; continuous monitoring is necessary to catch unfair decisions early.
Neglecting Model Retraining: Fairness should be considered during model training and validation phases.

#### Follow-Ups

How would you handle false positives in the audit tool?
What actions would you take if a fairness threshold is breached?

**Comparison:**

```
Team ID	Paged Incidents
1	1
2	1
3	1
```

**Code:**

```python
import json
from datetime import datetime, timedelta

def log_decision(team_id, decision):
    with open('decision_logs.json', 'a') as log_file:
        entry = {'team_id': team_id, 'decision': decision, 'timestamp': datetime.now().isoformat()}
        json.dump(entry, log_file)
        log_file.write('\n')

def audit_fairness(start_date=None, end_date=None):
    if start_date is None or end_date is None:
        now = datetime.now()
        start_date = now - timedelta(days=7)
        end_date = now

    log_file = open('decision_logs.json', 'r')
    total_paged = 0
    team_counts = {}

    for line in log_file:
        entry = json.loads(line.strip())
        if start_date <= datetime.fromisoformat(entry['timestamp']) < end_date and entry['decision'] == "page":
            team_id = entry['team_id']
            total_paged += 1
            if team_id not in team_counts:
                team_counts[team_id] = 0
            team_counts[team_id] += 1

    log_file.close()

    fairness_report = {
        'total_paged': total_paged,
        'team_frequencies': {team: count for team, count in team_counts.items()}
    }
    return fairness_report

def check_thresholds(report):
    threshold = report['total_paged'] * 0.8
    for team_id, count in report['team_frequencies'].items():
        if count > threshold:
            print(f"Team {team_id} is handling more than 80% of paged incidents.")

def train_model_with_fairness(X, y):
    X_train, X_val, y_train, y_val = train_test_split(X, y, test_size=0.2)

    model.fit(X_train, y_train)
    val_predictions = model.predict(X_val)

    # Evaluate fairness on validation data
    audit_fairness()

def monitor_fairness_periodically():
    while True:
        report = audit_fairness()
        check_thresholds(report)
        time.sleep(3600)  # Check every hour
```

**Example input / output:**

```

Sample input for log_decision and output from audit_fairness:

PYTHON
Copy
# Example data
team_ids = [1, 2, 3]  # Team IDs

for team_id in team_ids:
    log_decision(team_id, "page")

report = audit_fairness()
print(report)

Output:

JSON
Copy
{
    'total_paged': 3,
    'team_frequencies': {
        1: 1,
        2: 1,
        3: 1
    }
}
```

**Data:**

```sql
CREATE TABLE DecisionLog (
    id SERIAL PRIMARY KEY,
    team_id INT NOT NULL,
    decision VARCHAR(255) NOT NULL,
    timestamp TIMESTAMP NOT NULL
);
```

**Flow:**

1. Start
2. Log Decision
3. Record Log
4. Audit Fairness
5. Check Thresholds
6. Report Results
7. Monitor Continuously

