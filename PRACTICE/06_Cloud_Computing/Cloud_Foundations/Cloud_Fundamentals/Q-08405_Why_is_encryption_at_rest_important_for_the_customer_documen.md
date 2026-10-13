---
type: "practice"
difficulty: "Basic"
---

# Why is encryption at rest important for the customer documents your RAG system stores in cloud object storage?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Presigned URLs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Presigned_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Encryption at rest ensures that sensitive data, such as customer documents stored in a RAG (Relevant AI Generated) system, remains secure even if unauthorized access to the underlying infrastructure occurs. This is crucial because cloud object storage can be accessed by various parties, including cloud service providers and other entities within the cloud ecosystem.

To understand why encryption at rest is important, we need to consider both the security threats and the compliance requirements associated with storing sensitive data in the cloud.

**Comparison:**

| Step | Action                       | Description                                                                                         |
|------|-----------------------------|-----------------------------------------------------------------------------------------------------|
| 1    | Encrypt document            | Use an encryption algorithm with a unique key to transform the plaintext into ciphertext.           |
| 2    | Store in cloud              | Upload the encrypted data to cloud object storage for secure and accessible storage.                |
| 3    | Decrypt when needed         | Retrieve the encrypted data, apply the same decryption process using the key, and obtain the original data. |

**Code:**

```python
# Example of a simple encryption function (not for production use)
def encrypt_data(data):
    return bytes([b ^ 0xAA for b in data])

def decrypt_data(encrypted_data):
    return encrypt_data(encrypted_data)  # XOR with same key to decrypt
```

**Example input / output:**

```text
Input: Document data = "sensitive_customer_information"
Encryption key = "98f7d64e-8b00-4a5c-a0e0-7c1fafe21859"

Step 1: Encrypt the document data using the encryption key.
Encrypted data = encrypt_data("sensitive_customer_information", "98f7d64e-8b00-4a5c-a0e0-7c1fafe21859")

Step 2: Store the encrypted data in cloud object storage.

Step 3: When retrieving the document, use the same key to decrypt.
Decrypted data = decrypt_data(encrypted_data)
```

**Data:**

```json
{
  "documentId": "doc-123",
  "data": "sensitive_customer_information",
  "encryptionKey": "98f7d64e-8b00-4a5c-a0e0-7c1fafe21859"
}
```

**Flow:**

1. Yes
2. No
3. Store Document
4. Encrypt Data
5. Upload to Cloud Storage
6. Use Encryption Key
7. Securely Store EncryptedDocument
8. Decrypt Using Same Key
9. Retrieve Original Document

