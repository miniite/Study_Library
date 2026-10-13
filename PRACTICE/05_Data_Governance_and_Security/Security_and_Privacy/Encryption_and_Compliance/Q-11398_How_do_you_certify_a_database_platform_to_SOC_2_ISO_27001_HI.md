---
type: "practice"
difficulty: "Deep"
---

# How do you certify a database platform to SOC 2 / ISO 27001 / HITRUST on the encryption controls — what evidence do you automate and what's the continuous-compliance tooling?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Vendor Compliance Audits](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Vendor_Compliance_Audits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To certify a database platform for standards like SOC 2, ISO 27001, or HITRUST on the encryption controls, we need to ensure that our data is encrypted both in transit and at rest, and that these protections are well-documented and auditable. The process involves several key steps: understanding the requirements of the standard, implementing robust encryption mechanisms, and establishing automated evidence-gathering tools for continuous compliance.

The core idea here is to ensure that every aspect of data encryption is not only implemented but also monitored continuously to maintain compliance over time. This includes encrypting sensitive data, managing keys securely, auditing encryption operations, and ensuring that all these processes are documented and can be validated by external auditors.

**Comparison:**

```
Encryption Type	Usage	Compliance
In-transit	TLS	Yes
At-rest	AES-256	Yes
```

**Code:**

```python
# Example Python code for key management (simplified)
from cryptography.fernet import Fernet

class KeyManager:
    def generate_key(self):
        return Fernet.generate_key()

    def encrypt_data(self, data, key):
        f = Fernet(key)
        encrypted_data = f.encrypt(data.encode())
        return encrypted_data
```

**Example input / output:**

```

Input:

PLAINTEXT
Copy
User inputs sensitive data into the database.

Output (simplified):

PLAINTEXT
Copy
Data is encrypted using AES-256 with a generated key, and stored securely. Key management logs are updated to indicate that encryption was applied and by whom.
```

**Data:**

```json
{
  "encryption_standard": {
    "SOC2": true,
    "ISO27001": false,
    "HITRUST": false
  },
  "key_management": {
    "algorithm_used": "AES-256",
    "key_rotation_frequency": "every 90 days"
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Input
4. Is Data Sensitive?
5. Encrypt with AES-256
6. Store Encrypted Data Securely
7. Update Key Management Logs
8. Pass Through Unencrypted

