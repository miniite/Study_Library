---
type: "practice"
difficulty: "Deep"
---

# What is your strategy for handling encryption when adopting a managed DBaaS where you don't control the host — how do you retain key custody and prove the provider can't read PII?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Compliant Access to Encrypted PII](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Compliant_Access_to_Encrypted_PII.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In scenarios where managing sensitive data, particularly Personal Identifiable Information (PII), is critical, especially under regulations like GDPR or HIPAA, ensuring key custody and proving that your managed DBaaS provider cannot read PII without your explicit consent are paramount. My strategy involves a combination of robust encryption practices, key management systems (KMS), and careful contractual negotiations.

Firstly, the core idea is to use strong encryption methods that not only protect data but also ensure compliance with regulatory requirements. Secondly, we need a KMS where keys remain under our control, ensuring that even if there’s any breach or compromise at the provider's end, your data remains secure. Finally, stringent contractual terms must be agreed upon to outline the provider's responsibilities and limitations.

**Code:**

```python
# Example of encryption setup in Python using Fernet from cryptography library
from cryptography.fernet import Fernet

def generate_key():
    return Fernet.generate_key()

def encrypt_data(key, data):
    f = Fernet(key)
    encrypted = f.encrypt(data.encode())
    return encrypted

def decrypt_data(key, encrypted_data):
    f = Fernet(key)
    decrypted = f.decrypt(encrypted_data).decode()
    return decrypted
```

**Data:**

```json
{
  "key": "<generated_key>",
  "data": "Sensitive_PII"
}
```

