---
type: "practice"
difficulty: "Deep"
---

# Your pipeline posts status to PRs, but flaky required checks train reviewers to click re-run and merge; how do you keep gates meaningful and prevent alert fatigue?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Flaky Tests](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Flaky_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of maintaining meaningful gates in a CI/CD pipeline while preventing alert fatigue, we need to strike a balance between ensuring robustness in our builds and minimizing redundant notifications that can desensitize reviewers. The core idea is to refine the checks based on risk and importance, implement smarter notifications, and introduce adaptive mechanisms to handle flakiness.

#### Triaging the Question

The question revolves around optimizing CI/CD pipelines to ensure that only critical steps are gated, thereby reducing unnecessary re-runs and maintaining the quality of code reviews. The interviewer is probing for strategies to dynamically adjust the pipeline gates based on the context and recent history, as well as mechanisms to handle flaky tests effectively without overwhelming the team.

#### First-Principles Solution
Risk Assessment: Classify checks into critical and non-critical categories.
Flaky Test Handling: Implement intelligent retries with increasing delays for flaky tests.
Dynamic Gating: Use historical data to adaptively adjust which steps are gated based on their success rate.
Contextual Notifications: Notify the team only when necessary, with detailed information that helps in quick decision-making.
Step-by-Step Solution
1. Risk Assessment

First, categorize your pipeline checks into critical and non-critical steps. Critical steps might include tests that ensure core functionality or security. Non-critical steps could be linter runs or style checks. This classification can be done based on historical data or predefined business rules.

Critical Check
Build
Non-Critical Check
Linting
2. Flaky Test Handling

For flaky tests, implement retries with exponential backoff to avoid false negatives. This can be achieved by configuring your CI/CD tool to retry the test a specified number of times before failing.

Flaky Check
Try 1
Try 2
3. Dynamic Gating

Use machine learning models to predict the success rate of each step based on historical data. This can help in dynamically gating steps that are less likely to fail, reducing unnecessary re-runs.

PYTHON
import pandas as pd
from sklearn.ensemble import RandomForestClassifier

# Example dataset with features and labels
data = pd.read_csv('pipeline_history.csv')
X = data.drop(columns=['success'])
y = data['success']

model = RandomForestClassifier()
model.fit(X, y)

def predict_success(step_features):
    return model.predict_proba([step_features])[0][1]
4. Contextual Notifications

Implement smart notifications that notify the team only when necessary. Use inline comments or pull request status updates to provide context and explain why a particular step is being retried.

FLOWCHART · MERMAID
graph TD;
    A[Pipeline Start] --> B[Check Status]
    B[Check Status] --> C[Success] || D[Rerun Required]
    D[Rerun Required] --> E[Notify Team] (with detailed context)

#### Closing Summary
and Common Mistakes

#### Closing Summary

By classifying checks, handling flaky tests intelligently, dynamically gating based on historical data, and implementing smart notifications, we can keep our CI/CD pipeline gates meaningful while reducing alert fatigue. This approach ensures that critical steps are always protected without overwhelming the team with redundant notifications.

#### Common Mistakes
Over-gating: Keeping too many steps gated unnecessarily.
Ignoring Flakiness: Failing to handle flaky tests effectively, leading to unnecessary re-runs and alerts.
Lack of Feedback Loops: Not using historical data to continuously refine the gating strategy.
Inadequate Notifications: Sending notifications without providing enough context or explanations.

**Comparison:**

```
Test	Success Rate
TestA	0.98
TestB	0.5
```

**Code:**

```python
# Example code for dynamic gating using machine learning
import pandas as pd
from sklearn.ensemble import RandomForestClassifier

data = pd.read_csv('pipeline_history.csv')
X = data.drop(columns=['success'])
y = data['success']

model = RandomForestClassifier()
model.fit(X, y)

def predict_success(step_features):
    return model.predict_proba([step_features])[0][1]
```

**Example input / output:**

```text
graph TD;
    A[Pipeline Start] --> B[Check Status]
    B[Check Status] --> C[Success] || D[Rerun Required]
    D[Rerun Required] --> E[Notify Team] (with detailed context)
```

**Data:**

```
CSV
Copy
# Example dataset structure for pipeline history
,build_id,test_name,status,pipeline_step
1,Build 1,TestA,Success,Critical
2,Build 2,TestB,Fail,Non-Critical
3,Build 3,TestA,Success,Critical
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Pipeline Start] --> B[Check Status]
4. B[Check Status] --> C[Success] || D[Rerun Required]
5. D[Rerun Required] --> E[Notify Team] (with detailed context)

