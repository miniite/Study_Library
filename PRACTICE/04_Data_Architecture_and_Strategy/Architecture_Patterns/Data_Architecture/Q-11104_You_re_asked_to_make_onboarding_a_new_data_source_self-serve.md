---
type: "practice"
difficulty: "Intermediate"
---

# You're asked to make onboarding a new data source self-serve for other teams. What do you make mandatory at registration (schema, owner, PII flag, SLA) and how do you block ungoverned side-door ingestion?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Proving PII Residency](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Proving_PII_Residency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When onboarding a new data source to a self-serve environment, it's crucial to ensure that the process is governed, secure, and aligned with organizational standards. Key mandatory fields at registration include schema, owner, PII flag, and SLA. This setup helps in maintaining data integrity, accountability, and compliance while preventing ungoverned side-door ingestion.

#### Triage

The core of this question revolves around establishing a robust onboarding process for new data sources to ensure that they are properly governed within the organization's architecture (data lakes or warehouses). The interviewer is probing your understanding of governance practices, security considerations, and the importance of defining clear policies and workflows. Key assumptions include:

The system supports self-service registration.
Data ownership and accountability are critical.
Compliance with regulatory requirements for Personally Identifiable Information (PII) is mandatory.

#### Solution
Mandatory Fields at Registration
Schema: Defining a schema ensures that the data is structured correctly, adhering to standardized formats within the organization's architecture. This helps in integrating new sources seamlessly into existing systems.
Owner: Assigning ownership ensures accountability and responsibility for the data. The owner should be identified as a key stakeholder who can address any issues or concerns related to the data source.
PII Flag: Marking whether the data contains Personally Identifiable Information (PII) is crucial for compliance with data protection regulations such as GDPR, HIPAA, etc. This helps in managing access and handling sensitive information appropriately.
SLA (Service Level Agreement): Defining an SLA ensures that the data source meets performance and availability requirements. This includes response times, uptime guarantees, and backup/recovery policies.
Blocking Ungoverned Side-Door Ingestion

To block ungoverned side-door ingestion, several steps can be taken:

Pre-registration Validation: Implement a pre-registration validation process where new sources are reviewed by a governance team before they are allowed to ingest data into the system.
Automated Schema Validation: Integrate automated schema validation tools that check incoming data against the defined schema at registration. This ensures consistency and correctness.
Access Controls: Implement robust access controls and permissions management systems. Only authorized users should have the ability to register new sources, ensuring that all entries are compliant.
Monitoring and Auditing: Establish continuous monitoring and auditing processes to detect and address any unauthorized or non-compliant data ingestion attempts promptly.

#### Closing Summary

In summary, mandatory fields such as schema, owner, PII flag, and SLA at registration ensure a structured and governed environment for new data sources. Blocking ungoverned side-door ingestion involves a combination of pre-registration validation, automated schema checks, access controls, and robust monitoring mechanisms. This approach ensures compliance with organizational standards and regulatory requirements while maintaining data integrity and security.

#### Common Mistakes/Follow-Ups
Failing to Validate Schemas: Ensuring that all incoming data conforms to the defined schema is critical but often overlooked.
Overlooking PII Flags: Failing to properly flag PII can result in non-compliance with regulatory requirements, leading to potential legal and financial risks.
Neglecting SLA Definitions: Missing SLA definitions can lead to performance issues and downtime without clear service-level commitments.

#### Follow-up
questions might include discussing specific tools for schema validation, the role of DevOps in this process, or how to balance automation with manual oversight.

**Comparison:**

```
Step	Action	Responsible Party
1	Validate schema and other metadata	Automated Tools
2	Send approval email to owner	System
3	Ingest data after confirmation	Data Team
4	Reject request if invalid	System
```

**Code:**

```python
# Example Python script for automated schema validation

def validate_schema(data, expected_schema):
    if not all(key in data for key in expected_schema.keys()):
        return False
    for key, value_type in expected_schema.items():
        if not isinstance(data[key], value_type):
            return False
    return True
```

**Example input / output:**

```text
Input:
- New data source registration request with fields: schema, owner, PII flag, SLA

Output:
1. Pre-registration validation checks the schema and other metadata.
2. If valid, the system sends an approval email to the owner for final confirmation.
3. Once approved, the data can be ingested into the governed environment.
4. If invalid, the registration request is rejected with specific error messages.
```

**Data:**

```json
{
  "schema": {
    "customer_id": "integer",
    "name": "string",
    "email": "string",
    "address": "string"
  },
  "pii_flag": true,
  "owner": "DataTeamLead@acme.com",
  "sla": {
    "response_time": 30,
    "uptime": 99.9
  }
}
```

**Flow:**

1. Yes
2. No
3. New Data Source Registration
4. Is Schema Valid?
5. Send Approval Email
6. Ingest Data
7. Reject Request with ErrorMessages

