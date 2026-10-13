---
type: "practice"
difficulty: "Basic"
---

# Why do most AI providers require HTTPS for their REST APIs, and what does it protect when you send prompts and keys?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of URLs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Keeping_Secrets_out_of_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

HTTPS is widely used in REST APIs to ensure secure data transmission, primarily protecting sensitive information such as API keys, user credentials, and private data like prompts or input text. By using HTTPS, AI providers can prevent man-in-the-middle attacks and eavesdropping, thereby maintaining the confidentiality and integrity of the communication.

#### Triaging the Question

The question is probing your understanding of secure communication protocols in web services, specifically how HTTPS works to protect sensitive information during transmission. We need to assume that you have basic knowledge of HTTP vs HTTPS differences and common security threats in network communications.

#### First-Principles Solution

HTTPS vs HTTP:

HTTP (Hypertext Transfer Protocol): This is the standard protocol for transmitting data over a network, but it transmits data in plain text, meaning anyone who intercepts the data can read it.
HTTPS (HTTP Secure): This uses SSL/TLS (Secure Sockets Layer/Transport Layer Security) to encrypt the communication between the client and server. The encryption ensures that even if someone intercepts the data, they cannot read or tamper with it.

Encryption:

Data Integrity: HTTPS ensures that any modifications to the data during transmission are detected because the integrity of the encrypted payload is verified.
Confidentiality: The use of encryption prevents unauthorized access and eavesdropping on sensitive information like API keys or user credentials.

Authentication:

Both HTTP and HTTPS provide client authentication through certificates, but HTTPS also provides server authentication to ensure that the communication is with the correct server.

Common Security Threats:

Man-in-the-Middle (MitM) Attacks: In an MitM attack, an attacker intercepts the communication between the client and server. HTTPS prevents this by ensuring that any intercepted data cannot be read or modified.
Session Hijacking: HTTPS ensures session integrity and prevents hijacking because it encrypts session cookies.

#### Closing Summary

In summary, HTTPS is crucial for securing REST APIs in AI services by encrypting data to prevent eavesdropping and tampering. It ensures the confidentiality and integrity of communications, making it a best practice for transmitting sensitive information such as API keys and user prompts securely over networks.

#### Common Mistakes/Follow-Ups
Assuming All HTTPS Connections Are Secure: Sometimes, poorly configured TLS settings or weak encryption can leave connections vulnerable.
Not Understanding the Difference Between HTTP vs HTTPS: Some developers might not fully grasp why HTTPS is necessary when HTTP works fine for non-sensitive data.

**Code:**

```python
# Example of making a secure API call using Python's requests library with HTTPS
import requests

response = requests.get('https://api.example.com/v1/data', auth=('username', 'password'))
print(response.json())
```

**Example input / output:**

```text
No worked input->output example is necessary here, as the focus is on explaining concepts rather than concrete interactions.
```

**Flow:**

1. Client
2. HTTPS Server
3. Encrypted Data
4. Server
5. Decrypted Data
6. API Response

