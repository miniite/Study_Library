---
type: "practice"
difficulty: "Deep"
---

# How do you govern the emergency-access break-glass procedure to sensitive data during an incident so it's usable under pressure yet fully audited afterward?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Preventing PII Leakage](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Preventing_PII_Leakage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with emergency access to sensitive data, we must strike a balance between rapid response and comprehensive auditing. This involves establishing clear procedures, roles, and tools that ensure authorized personnel can access critical information swiftly while maintaining a robust audit trail. The core idea is to create a controlled environment where only the necessary individuals have access under extreme circumstances.

This question requires us to consider several aspects: defining the emergency-access protocol, ensuring its usability during incidents, and guaranteeing thorough auditing post-incident. We'll break it down step by step from first principles, covering roles, technologies, and best practices.

First, we need to define the roles involved in this process. Typically, these include:

Incident Response Team (IRT): Trained individuals who can make immediate decisions during an incident.
Data Governance Officers: Oversee data access policies and audit procedures.
Access Review Board (ARB): A group authorized to approve emergency access requests.

Next, we'll establish the technology stack:

Break-Glass Authentication Tokens: These tokens allow for quick authentication bypass in emergencies.
Real-Time Monitoring Tools: To track and log all access attempts.
Audit Logs Database: Stores a detailed history of every action taken during an incident.

Let's walk through each step:

Define the Break-Glass Protocol:

Token Generation: Create unique, time-limited tokens that can be used to bypass normal authentication mechanisms.
Access Approval: ARB members must approve the token generation and distribution. This ensures only authorized individuals have access.

Implement Real-Time Monitoring:

Log All Access Attempts: Record every interaction with sensitive data, including timestamps, user actions, and any modifications made.
Alert Mechanisms: Set up real-time alerts for unusual activity or repeated login attempts from unauthorized users.

Maintain Comprehensive Audit Trails:

Post-Incident Review: Conduct thorough reviews of audit logs to ensure no sensitive data was misused or altered.
Automated Reports: Generate automated reports summarizing the incident, key actions taken, and any potential vulnerabilities identified.

Training and Drills:

Regularly train IRT members on how to use the break-glass protocol effectively.
Conduct regular drills to test the emergency access procedures under simulated incident conditions.

To implement this solution, we might design a database schema like this:

SQL
CREATE TABLE EmergencyAccessLogs (
    log_id INT PRIMARY KEY,
    user_id VARCHAR(255) NOT NULL,
    action TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    resource_type VARCHAR(100),
    resource_name VARCHAR(255),
    action_details TEXT
);

We would also develop a Python script to handle token generation and validation:

PYTHON
import datetime

def generate_token(user_id, expiration_time):
    # Generate a unique token for the given user
    token = f"{user_id}_{datetime.datetime.now().timestamp()}"

    return token

def validate_token(token, user_id):
    # Validate if the token is still valid and belongs to the user
    parts = token.split('_')
    if len(parts) != 2:
        return False

    stored_user_id, timestamp_str = parts
    if stored_user_id == user_id and datetime.datetime.fromtimestamp(float(timestamp_str)) > datetime.datetime.now():
        return True

    return False

These scripts would be integrated into our monitoring system to handle emergency access requests.

In conclusion, a well-structured break-glass procedure for sensitive data must balance speed with security. By clearly defining roles, leveraging appropriate technologies, and maintaining detailed audit trails, we can ensure that critical data is accessible during emergencies while providing robust post-incident oversight.

#### Common mistakes
include:

Failing to establish clear authorization protocols.
Overlooking the importance of real-time monitoring and alerts.
Not conducting regular training and drills to ensure readiness.

**Comparison:**

| log_id | user_id     | action            | resource_type | resource_name       | action_details                                        |
|--------|-------------|-------------------|---------------|--------------------|------------------------------------------------------|
| 1      | john_doe    | 2021-09-25 14:00  | customer_data | John Doe's Records  | Accessed customer data for troubleshooting           |

**Code:**

```python
import datetime

def generate_token(user_id, expiration_time):
    # Generate a unique token for the given user
    token = f"{user_id}_{datetime.datetime.now().timestamp()}"

    return token

def validate_token(token, user_id):
    # Validate if the token is still valid and belongs to the user
    parts = token.split('_')
    if len(parts) != 2:
        return False

    stored_user_id, timestamp_str = parts
    if stored_user_id == user_id and datetime.datetime.fromtimestamp(float(timestamp_str)) > datetime.datetime.now():
        return True

    return False
```

**Example input / output:**

```text
Request:
User 'john_doe' attempts to access sensitive data with token 'johndoe_1633072800'

Response (log entry):
log_id: 1, user_id: 'john_doe', action: '2021-09-25 14:00:00', resource_type: 'customer_data', resource_name: 'John Doe's Records', action_details: 'Accessed customer data for troubleshooting'
```

**Data:**

```sql
CREATE TABLE EmergencyAccessLogs (
    log_id INT PRIMARY KEY,
    user_id VARCHAR(255) NOT NULL,
    action TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    resource_type VARCHAR(100),
    resource_name VARCHAR(255),
    action_details TEXT
);
```

**Flow:**

1. Initiate Incident
2. Verify IRT Members
3. Generate Break-Glass Token
4. ARBoard Approve
5. Authenticate with Token
6. Audit Logs Created
7. Post-Incident Review

