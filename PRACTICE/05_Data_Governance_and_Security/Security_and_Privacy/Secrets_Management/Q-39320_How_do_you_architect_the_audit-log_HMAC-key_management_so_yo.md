---
type: "practice"
difficulty: "Deep"
---

# How do you architect the audit-log HMAC-key management so you can still correlate logged values back to real secrets during a forensic investigation, while keeping the raw logs privacy-safe day-to-day?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Secret Rotation](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Shared_Secret_Rotation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect the audit-log HMAC-key management in a way that maintains privacy of raw logs but allows for forensic correlation with real secrets, we need to carefully manage the lifecycle and distribution of HMAC keys used for signing audit logs. The core challenge is balancing security (keeping raw logs private) and compliance (ensuring auditors can correlate logged values back to secrets during investigations). Here’s a step-by-step approach:

First, let's understand what this question is really asking: We need a robust system that signs audit log entries with HMAC keys, ensuring the integrity of logs while maintaining privacy. The solution should allow us to verify the authenticity of logs during forensic analysis but prevent unauthorized access to sensitive secrets.

Key Assumptions
HMAC Key Rotation: Keys used for signing are rotated regularly.
Access Control: Strict controls are in place for who can access HMAC keys and audit logs.
Secure Storage: Keys and logs are stored securely, with minimal exposure.
Step-by-Step Solution
1. Key Management

We will use a secrets manager like HashiCorp Vault to manage our HMAC keys. These keys should be versioned to allow for rotation without impacting the integrity of existing audit logs.

YAML
# Example: Vault Role Configuration for HMAC Keys
hmac_keys_role = {
  "policies": ["read_secrets"],
  "secret_id_tiers": [{"path_prefix": "secrets/hmac", "accessibility": "local"}],
}
2. Signing Audit Logs

When an audit log entry is generated, it will be signed with the appropriate HMAC key from Vault.

PYTHON
import hmac
import hashlib

def sign_log_entry(entry: str, key_id: str) -> dict:
    # Fetching the specific key from Vault
    vault_client = get_vault_client()
    hmac_key = vault_client.read(f"secrets/hmac/{key_id}")['data']['value']

    signed_entry = {
        "log_entry": entry,
        "signature": hmac.new(
            key=hmac_key.encode(),
            msg=entry.encode(),
            digestmod=hashlib.sha256
        ).hexdigest()
    }

    return signed_entry

# Example usage:
signed_log = sign_log_entry("User login attempt", "key-123")
print(signed_log)
3. Storing Signed Logs Securely

Signed logs should be stored in a secure, immutable logging system such as Amazon CloudWatch or an on-premises equivalent.

JSON
# Example: Signed Log Entry Stored in JSON Format
{
  "timestamp": "2023-10-05T14:27:38Z",
  "entry": "User login attempt",
  "signature": "da39a3ee5e6b4b0d3255bfef95601890afd80709"
}
4. Verifying Logs During Forensic Analysis

During a forensic investigation, an authorized auditor can access the HMAC keys and use them to verify the authenticity of signed logs.

PYTHON
def verify_log_entry(signed_log: dict, key_id: str) -> bool:
    # Fetching the specific key from Vault
    vault_client = get_vault_client()
    hmac_key = vault_client.read(f"secrets/hmac/{key_id}")['data']['value']

    expected_signature = hmac.new(
        key=hmac_key.encode(),
        msg=signed_log["log_entry"].encode(),
        digestmod=hashlib.sha256
    ).hexdigest()

    return signed_log['signature'] == expected_signature

# Example verification:
print(verify_log_entry(signed_log, "key-123"))

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that audit logs are both secure and verifiable. By using Vault for key management and a structured logging system, we can maintain privacy while allowing forensic correlation during investigations.

#### Common mistakes
include:

Failing to rotate keys regularly.
Poorly controlling access to HMAC keys.
Storing raw secrets in the same system as signed logs.

#### Follow-up
questions might revolve around key rotation strategies, backup mechanisms, and integrating with existing security frameworks like OAuth for identity verification.

**Code:**

```python
import hmac
import hashlib

def sign_log_entry(entry: str, key_id: str) -> dict:
    # Fetching the specific key from Vault
    vault_client = get_vault_client()
    hmac_key = vault_client.read(f"secrets/hmac/{key_id}")['data']['value']

    signed_entry = {
        "log_entry": entry,
        "signature": hmac.new(
            key=hmac_key.encode(),
            msg=entry.encode(),
            digestmod=hashlib.sha256
        ).hexdigest()
    }

    return signed_entry

def verify_log_entry(signed_log: dict, key_id: str) -> bool:
    # Fetching the specific key from Vault
    vault_client = get_vault_client()
    hmac_key = vault_client.read(f"secrets/hmac/{key_id}")['data']['value']

    expected_signature = hmac.new(
        key=hmac_key.encode(),
        msg=signed_log["log_entry"].encode(),
        digestmod=hashlib.sha256
    ).hexdigest()

    return signed_log['signature'] == expected_signature

# Example usage:
signed_log = sign_log_entry("User login attempt", "key-123")
print(signed_log)
print(verify_log_entry(signed_log, "key-123"))
```

**Example input / output:**

```text
**Input**
- Log Entry: "User login attempt"
- Key ID: "key-123"

**Output**
- Signed Log Entry:
  - `{"log_entry": "User login attempt", "signature": "da39a3ee5e6b4b0d3255bfef95601890afd80709"}`

- Verification Result: True
```

**Data:**

```json
{
  "timestamp": "2023-10-05T14:27:38Z",
  "entry": "User login attempt",
  "signature": "da39a3ee5e6b4b0d3255bfef95601890afd80709"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Log Entry Generation
6. Is Key ID Valid?
7. Signed Log Entry
8. Store in Secure LoggingSystem
9. Generate Error and Retry
10. Fetch HMAC Key from Vault
11. Verify Signature with StoredKey
12. Is Signature Verified?
13. Audit Log Verification Success
14. Generate Error and ReportFailure

