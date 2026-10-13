---
type: "practice"
difficulty: "Deep"
---

# Why is 'grant access then review usage later' sometimes the right governance posture versus strict up-front denial, and how do you build the detective controls that make permissive-by-default safe?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing User-owned Transformations](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Governing_User-owned_Transformations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data security, the principle of "grant access then review usage later" (permit-by-default) can be more effective than strict up-front denial, especially in environments where data is highly sensitive and complex access patterns are hard to predict upfront. This approach allows for a more fluid and adaptable data environment but requires robust detective controls to ensure that unauthorized access does not occur. Here’s why this posture works and how you can implement the necessary detective controls.

First, let's break down the question: The core of your task is understanding under what circumstances granting access first is preferred over denying it upfront, and then discussing the mechanisms needed to monitor and control usage effectively.

#### Triaging the Question
Context: This approach is particularly relevant in large enterprises where data is critical but not fully understood until accessed and used.

#### Assumptions
Data sensitivity varies greatly within an organization.
There's a need for rapid prototyping and experimentation with data.
Key Concepts:
Permissive access: Granting access by default to allow users to explore data freely.
Detective controls: Implementing mechanisms to detect and respond to unauthorized or suspicious activity.

#### First-Principles Solution

Why Permit-by-Default is Preferred:

Faster Time-to-Value: Users can start exploring the data immediately, accelerating insights and decision-making processes.
Innovation: Encourages experimentation and innovation without upfront approval bottlenecks.
Data Discovery: Facilitates more comprehensive data understanding as users interact with the data.

Detective Controls:

Logging and Monitoring: Implement detailed logging of all access and usage activities to track who is accessing what, when, and how.
Anomaly Detection Systems: Use machine learning models to identify unusual patterns or deviations from normal behavior indicative of misuse.
Role-Based Access Control (RBAC): Ensure that roles are finely tuned to reflect specific responsibilities and permissions.
Audit Trails: Maintain comprehensive audit trails that can be used for forensic analysis if issues arise.
Building Detective Controls

Logging Setup:

SQL
-- Example of a basic logging schema in a SQL database
CREATE TABLE AccessLogs (
    id SERIAL PRIMARY KEY,
    userId INT REFERENCES Users(id),
    datasetId INT REFERENCES Datasets(id),
    action VARCHAR(255),  -- e.g., read, write, delete
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Anomaly Detection:

PYTHON
# Example of a simple anomaly detection model using pandas and scikit-learn
import pandas as pd
from sklearn.ensemble import IsolationForest

def detect_anomalies(logs_df):
    X = logs_df[['userId', 'datasetId', 'action']]
    model = IsolationForest(contamination=0.01)  # Adjust contamination based on historical data
    model.fit(X)
    anomalies = model.predict(X)
    return anomalies, X

RBAC Implementation:

JSON
// Example RBAC policy for a dataset
{
  "datasetId": 123,
  "roles": [
      {"roleName": "Analyst", "permissions": ["read", "write"]},
      {"roleName": "Developer", "permissions": ["read"]}
  ]
}

#### Closing Summary
and Common Mistakes
Summary: Permit-by-default with strong detective controls is a pragmatic approach that balances operational flexibility with security. The key lies in robust logging, real-time anomaly detection, and fine-grained access policies.

#### Common Mistakes
Failing to implement detailed logging mechanisms can leave blind spots in detecting unauthorized activities.
Over-reliance on upfront approvals can stifle innovation and slow down data usage.
Not using machine learning for dynamic threat detection can result in missed anomalies.

**Comparison:**

```
userId	datasetId	action
1	123	read
2	456	write
```

**Code:**

```

CREATE TABLE AccessLogs ( id SERIAL PRIMARY KEY, userId INT REFERENCES Users(id), datasetId INT REFERENCES Datasets(id), action VARCHAR(255), -- e.g., read, write, delete timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP );

CODE
Copy

```python
import pandas as pd
from sklearn.ensemble import IsolationForest

def detect_anomalies(logs_df):
    X = logs_df[['userId', 'datasetId', 'action']]
    model = IsolationForest(contamination=0.01)  # Adjust contamination based on historical data
    model.fit(X)
    anomalies = model.predict(X)
    return anomalies, X
```

**Example input / output:**

```

Sample input:

JSON
Copy
[
    {"userId": 1, "datasetId": 123, "action": "read"},
    {"userId": 2, "datasetId": 456, "action": "write"}
]

Sample output from logging setup:

SQL
Copy
INSERT INTO AccessLogs (userId, datasetId, action) VALUES (1, 123, 'read');
INSERT INTO AccessLogs (userId, datasetId, action) VALUES (2, 456, 'write');
```

**Data:**

```json
{
  "datasetId": 123,
  "roles": [
      {"roleName": "Analyst", "permissions": ["read", "write"]},
      {"roleName": "Developer", "permissions": ["read"]}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Requests Access
6. Permit Request
7. Is Permission Approved?
8. Access Granted
9. Log Attempt
10. User Uses Data
11. Access Logs Matched withRBAC?
12. Audit Passes
13. Anomaly Detected

