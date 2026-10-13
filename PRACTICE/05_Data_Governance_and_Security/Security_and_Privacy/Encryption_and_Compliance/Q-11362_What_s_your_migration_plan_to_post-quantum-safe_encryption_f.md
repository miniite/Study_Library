---
type: "practice"
difficulty: "Expert"
---

# What's your migration plan to post-quantum-safe encryption for long-lived PII that must stay confidential for 30+ years — how do you crypto-agility-proof the schema today?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Compliant Access to Encrypted PII](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Compliant_Access_to_Encrypted_PII.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need for a post-quantum-safe encryption migration plan to ensure the confidentiality of long-lived Personally Identifiable Information (PII) for 30+ years, we must implement a robust and flexible approach that adheres to compliance standards like GDPR and HIPAA. The key steps include assessing current encryption methods, implementing modern cryptographic algorithms, and designing a schema that supports crypto-agility.

First, assess the current state of encryption in your database. Identify which columns contain sensitive PII and assess their current encryption methods. Understand the lifecycle of data encryption keys, including key rotation policies and storage practices.

Next, implement post-quantum-safe algorithms such as Lattice-based cryptography or code-based cryptosystems. These are designed to resist attacks from quantum computers and thus provide long-term security guarantees. Ensure that these new algorithms are integrated into your existing database schema without disrupting operations.

Then, design a schema that supports crypto-agility. This means structuring the data in a way that allows for seamless updates to encryption methods over time. For example, you can use separate columns for sensitive data and corresponding encrypted data, ensuring that changes in encryption algorithms do not require extensive database refactoring.

Finally, establish robust key management practices. Use HSMs (Hardware Security Modules) or cloud-based Key Management Services (KMS) to securely store and manage keys. Implement strict access controls and audit trails for key operations.

**Comparison:**

| Column | Data Type | Purpose |
|--------|----------|---------|
| id     | INTEGER  | Unique identifier for each record |
| name   | TEXT     | Name of the individual |
| ssn    | TEXT     | Social Security Number (PII) |
| encrypted_data | TEXT | Encrypted PII using post-quantum-safe algorithms |

**Code:**

```sql
-- Example SQL for adding a column with post-quantum-safe encryption
ALTER TABLE personal_info ADD COLUMN encrypted_data TEXT;

-- Example of a stored procedure for updating data with new encryption
CREATE OR REPLACE FUNCTION update_encrypted_data(new_value TEXT, key_id INTEGER)
RETURNS VOID AS $$
BEGIN
    UPDATE personal_info SET encrypted_data = encrypt(new_value, key_id);
END;
$$ LANGUAGE plpgsql;
```

**Example input / output:**

```text
Request:
{
  "id": 1,
  "new_value": "updated_info",
  "key_id": 12345
}

Response:
{
  "status": "success",
  "message": "Data updated successfully"
}
```

**Data:**

```json
{
  "personal_info": [
    {
      "id": 1,
      "name": "John Doe",
      "ssn": "123-45-6789",
      "encrypted_data": "encrypted_value"
    }
  ]
}
```

**Flow:**

1. Assess Current Encryption
2. Identify Sensitive Columns
3. ImplementPost-Quantum-SafeAlgorithms
4. Design Schema forCrypto-Agility
5. Establish Key ManagementPractices
6. Monitor and Update Regularly

