---
type: "practice"
difficulty: "Deep"
---

# What governance controls would you put in place so that PII does not leak from a production OLTP database into analytics, ML feature stores, and lower environments?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Preventing PII Leakage](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Preventing_PII_Leakage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that Personally Identifiable Information (PII) does not leak from a production Online Transaction Processing (OLTP) database into analytics, machine learning (ML) feature stores, and lower environments, we need to implement a robust data governance framework. This involves multiple layers of control, ranging from technical and procedural measures to regulatory compliance. The key steps are as follows:

Data Masking: Implement strong data masking techniques to obfuscate sensitive PII in non-production environments.
Access Controls: Establish strict access controls based on the principle of least privilege (PoLP) to ensure that only authorized personnel can interact with sensitive data.
Encryption: Use encryption at rest and in transit to protect PII from unauthorized access, whether it's being stored or transmitted.
Data Lineage Tracking: Maintain a clear understanding of how data flows through the system using data lineage tracking tools.
Automated Audits and Logging: Regularly audit data usage and log all interactions with sensitive information for compliance and forensic purposes.

#### Triaging the Question

The question is asking about comprehensive governance controls to prevent PII from leaking into unintended environments. The interviewer likely wants to see your understanding of best practices in data security, access management, and technical controls. Key assumptions include regulatory compliance (e.g., GDPR, HIPAA), the need for cross-functional team involvement, and the importance of automated tools.

Step-by-Step Solution
1. Data Masking

Data masking is essential to protect sensitive information while still allowing it to be used in development and testing environments. Techniques such as randomization, tokenization, or substitution can be employed to replace actual PII with a version that preserves the structure but not the meaning.

2. Access Controls

Implement role-based access control (RBAC) systems where users are granted permissions based on their roles within the organization. This ensures that only those who need access to PII have it, and even then, in the least privileged manner possible. Additionally, enforce multi-factor authentication (MFA) for critical operations.

3. Encryption

Use strong encryption standards like AES-256 both at rest and in transit. Ensure that all sensitive data is encrypted before it leaves the OLTP environment and when stored in lower environments or feature stores.

4. Data Lineage Tracking

Implement a data governance platform to track how PII flows through different systems. This helps in understanding dependencies, identifying potential leakage points, and ensuring compliance with data protection policies. Tools like Apache Atlas or Data Catalogs can be leveraged for this purpose.

5. Automated Audits and Logging

Set up comprehensive logging mechanisms that capture all interactions with sensitive data. These logs should be regularly audited to ensure compliance and detect any unauthorized access attempts. Implement automated alerting systems to notify security teams of suspicious activity in real-time.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Implementing these governance controls requires a holistic approach, involving technical, operational, and organizational elements. The primary challenges include ensuring that all stakeholders understand their roles, integrating these measures into existing workflows without disrupting operations, and keeping up with evolving regulatory requirements. Regular reviews and updates to the data governance framework are essential to maintain its effectiveness.

#### Common mistakes
often include underestimating the complexity of the implementation, neglecting regular audits and updates, and failing to involve cross-functional teams in the design process. Follow-ups could explore specific tools or technologies for data masking, encryption standards, and data lineage tracking systems.

**Comparison:**

| Data Element | Masking Technique          |
|--------------|----------------------------|
| First Name   | Initials + Random Characters |
| Last Name    | Initials + Random Characters |
| Email        | Domain Hiding, Randomized TLD |
| Phone Number | Obfuscated with random digits |

**Code:**

```sql
-- Example SQL Masking Function
CREATE OR REPLACE FUNCTION mask_pii(column_value VARCHAR)
RETURNS VARCHAR AS $$
BEGIN
    IF LENGTH(column_value) > 10 THEN
        RETURN SUBSTRING(column_value FROM 1 FOR 5) || '****' || SUBSTRING(column_value FROM -4);
    ELSE
        RETURN '******';
    END IF;
END;
$$ LANGUAGE plpgsql;
```

**Example input / output:**

```text
Input:
customer_id: 1, first_name: John, last_name: Doe, email: john.doe@example.com, phone_number: (555) 123-4567

Output after masking:
customer_id: 1, first_name: Jo****e, last_name: Do***e, email: **ohn.doe@example.com, phone_number: *****(123)-45**
```

**Data:**

```json
{
    "schema": {
        "customer_id": "INT",
        "first_name": "VARCHAR(50)",
        "last_name": "VARCHAR(50)",
        "email": "VARCHAR(100)",
        "phone_number": "VARCHAR(20)"
    },
    "example_data": [
        {"customer_id": 1, "first_name": "John", "last_name": "Doe", "email": "john.doe@example.com", "phone_number": "(555) 123-4567"},
        {"customer_id": 2, "first_name": "Jane", "last_name": "Smith", "email": "jane.smith@example.com", "phone_number": "(555) 987-6543"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Production OLTP DB
6. Data Masking?
7. Masked Data
8. Dev/Test Environments
9. Unmasked Data
10. Data Lineage Tracking?
11. Tracked Data
12. Audits & Logging
13. No Monitoring
14. Security Breach Risk

