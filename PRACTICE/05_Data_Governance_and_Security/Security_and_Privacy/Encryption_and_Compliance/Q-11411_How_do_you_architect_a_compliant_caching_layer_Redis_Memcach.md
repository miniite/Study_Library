---
type: "practice"
difficulty: "Deep"
---

# How do you architect a compliant caching layer (Redis/Memcached) that holds decrypted PII for performance — TTL, eviction, and memory-scraping risk versus encrypting cache contents?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Compliant Access to Encrypted PII](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Encryption_and_Compliance/Compliant_Access_to_Encrypted_PII.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a compliant caching layer using Redis or Memcached while holding decrypted Personally Identifiable Information (PII) for performance reasons, we need to balance several critical factors: compliance with regulations like GDPR and HIPAA, managing data at rest and in transit, ensuring secure eviction policies, and minimizing the risk of memory scraping. Let's break this down step by step.

Firstly, it’s essential to understand that decrypting PII and storing it in a caching layer poses significant risks, as these systems are not designed for long-term storage of sensitive data. However, if there is a compelling performance requirement, we need to implement robust safeguards.

#### Triaging the Question

The core issue here revolves around ensuring compliance while optimizing cache performance. We need to:

Ensure that decrypted PII is stored securely.
Implement proper eviction policies and Time-To-Live (TTL) settings.
Protect against memory scraping attacks.
Regularly review and update security measures.

#### Assumptions
The application has a valid reason for storing decrypted PII in the cache, such as high read traffic or complex data processing pipelines.
Compliance with GDPR/HIPAA is non-negotiable.
We are using Redis/Memcached, and both support encryption at rest and in transit.

#### Solution

Data Encryption: Encrypt PII before storing it in the cache. Use strong encryption algorithms like AES-256, ensuring that only authorized systems can decrypt the data.

Secure Storage: Ensure that the encrypted data is stored securely on disk using filesystem-level encryption or at the database level (if Redis/Memcached supports such features). For example, AWS EBS volumes with encryption enabled.

Memory Scrape Protection:

Use a non-volatile cache solution like Redis on Red Hat OpenShift, which has built-in mechanisms to prevent memory scrapers from accessing sensitive data.
Implement network isolation and use secure channels (TLS) for data transfer between the application and the cache server.

Eviction Policies:

Set appropriate TTL values based on the risk profile of the PII. For high-risk data, consider short TTLs to minimize exposure time.
Evict old or rarely accessed entries more aggressively to ensure that sensitive data does not linger in the cache longer than necessary.

Access Controls:

Implement strict access controls using RBAC (Role-Based Access Control) to limit who can read from and write to the cache.
Monitor and audit access logs regularly.

Regular Audits: Conduct regular security audits and compliance checks to ensure that all security measures are up-to-date and effective.

#### Closing Summary

By implementing a combination of strong encryption, secure storage practices, robust eviction policies, and strict access controls, we can architect a compliant caching layer using Redis/Memcached for decrypted PII. However, it's crucial to continuously review and update these measures as new threats emerge or regulatory requirements change.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to encrypt data at all points (both in transit and at rest). Ensure that encryption is applied uniformly.

#### Follow-Up
Discuss how specific tools like AWS KMS can be used for key management, or how hardware security modules (HSMs) can provide additional layers of security.

**Comparison:**

| Eviction Policy | Description                     |
|-----------------|---------------------------------|
| Least Recently Used (LRU) | Evicts the least recently used items first. |
| Time-to-Live (TTL)         | Evicts entries that have exceeded their TTL. |

**Code:**

```python
# Example Python code snippet for encrypting and decrypting data with AES256

from Crypto.Cipher import AES
import base64
import os

def pad(s):
    return s + (16 - len(s) % 16) * chr(16 - len(s) % 16)

def encrypt(raw, iv):
    raw = pad(raw)
    cipher = AES.new(iv, AES.MODE_CBC, iv)
    return base64.b64encode(cipher.encrypt(raw))

def decrypt(enc, iv):
    enc = base64.b64decode(enc)
    cipher = AES.new(iv, AES.MODE_CBC, iv)
    return cipher.decrypt(enc).rstrip(chr(16) * (enc[-1] if isinstance(enc, bytes) else ord(enc[-1])))
```

**Example input / output:**

```text
Input (Decrypted PII): {"name": "John Doe", "email": "johndoe@example.com"}
Output (Encrypted PII with TTL):
- Key: "john_doe_01"
- Value: "encrypted_data_here"
- TTL: 3600 seconds
```

**Data:**

```yaml
# Sample YAML configuration for Redis with TLS enabled
redis:
  host: redis.example.com
  port: 6379
  username: cache_user
  password: super_secret_password
  ssl: true
  tls_version: TLSv1.2
```

**Flow:**

1. No
2. Yes
3. Security Measures
4. Secure Storage
5. Network Isolation
6. Access Controls
7. Audit Logs
8. User Request
9. Cache Hit?
10. Fetch from Database
11. Encrypt Data
12. Store in Cache
13. Set TTL and Eviction Policy
14. Return Cached Data

