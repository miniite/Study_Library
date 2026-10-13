---
type: "practice"
difficulty: "Expert"
---

# Design the incident-response data flow so that when a breach is detected you can prove within GDPR's 72-hour window exactly which encrypted records were and weren't exposed.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Vendor Compliance Audits](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Vendor_Compliance_Audits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure compliance with GDPR's 72-hour breach notification requirement, we need to design an incident-response data flow that accurately tracks and reports on encrypted records. The core idea is to maintain a detailed audit trail of access attempts and outcomes, which can be used to demonstrate what data was potentially exposed during the breach. This involves several steps: detecting breaches, logging relevant information, encrypting sensitive fields if necessary, and ensuring rapid, verifiable reporting.

The question is probing our understanding of how to implement robust security measures while maintaining compliance with regulatory standards. We need to consider factors such as encryption methods, access controls, logging mechanisms, and data validation processes. Additionally, we must ensure that the system can quickly generate a comprehensive report to meet GDPR's stringent requirements.

First-principles reasoning starts by breaking down the problem into its core components:

Incident Detection: Detecting unauthorized access or breaches.
Logging Mechanisms: Recording detailed logs of access attempts and outcomes.
Data Encryption: Ensuring that sensitive data is encrypted both at rest and in transit.
Compliance Reporting: Generating a report within 72 hours to prove which records were potentially exposed.

To implement this, we will follow these steps:

Set up a robust incident detection system using mechanisms like intrusion detection systems (IDS) or security event management tools (SIEM).
Implement comprehensive logging for all access attempts to sensitive data.
Ensure proper encryption practices, including key management and secure storage of encrypted data.
Develop a reporting mechanism that can quickly generate the necessary information within 72 hours.

The solution involves creating an architecture where each component works seamlessly to ensure compliance and rapid response:

Incident Detection: Use SIEM tools to monitor logs for unusual activity.
Logging: Implement fine-grained logging for access attempts, including timestamps, user IDs, and actions taken.
Data Encryption: Ensure that sensitive fields are encrypted using strong encryption algorithms like AES-256 with proper key management practices.
Compliance Reporting: Develop a reporting system that can quickly generate an audit trail of who accessed which data.

By following this approach, we ensure that in the event of a breach, we have a clear and verifiable record of what records were potentially exposed. This not only meets GDPR's requirements but also provides a robust security framework for other regulatory compliances as well.

#### Closing summary
The key to designing an incident-response data flow is to create a system that can detect breaches quickly, log all relevant activities comprehensively, encrypt sensitive data effectively, and generate reports swiftly to demonstrate compliance. Common mistakes include overlooking proper logging mechanisms, inadequate encryption practices, and not having a streamlined reporting process in place.

**Comparison:**

```
AuditLog ID	User ID	Action	Timestamp	Encrypted Record ID
1	admin	viewed record	2023-10-15T14:30:00Z	encrypted-123
2	admin	modified record	2023-10-15T14:45:00Z	encrypted-456
3	admin	viewed record	2023-10-15T14:50:00Z	encrypted-789
```

**Code:**

```

CREATE TABLE AuditLog ( id INT PRIMARY KEY, user_id VARCHAR(255), action VARCHAR(100), timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP, encrypted_record_id VARCHAR(255) );

CODE
Copy

```python
# Example of a Python function to log access attempts
def log_access_attempt(user_id, action, record_id):
    with open('audit_log.txt', 'a') as f:
        f.write(f"{user_id},{action},{record_id},{datetime.now()}\n")
```

**Example input / output:**

```text
Input:
User ID: admin, Action: viewed record

Output:
AuditLog entry created with id=3, user_id=admin, action=viewed record, timestamp=2023-10-15T14:50:00Z, encrypted_record_id=encrypted-789
```

**Data:**

```json
{
  "AuditLog": [
    {
      "id": 1,
      "user_id": "admin",
      "action": "viewed record",
      "timestamp": "2023-10-15T14:30:00Z",
      "encrypted_record_id": "encrypted-123"
    },
    {
      "id": 2,
      "user_id": "admin",
      "action": "modified record",
      "timestamp": "2023-10-15T14:45:00Z",
      "encrypted_record_id": "encrypted-456"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Incident Detection
4. Is there an anomaly?
5. Audit Log Entry
6. Continue Monitoring
7. Generate Report
8. Compliance Check

