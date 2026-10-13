---
type: "practice"
difficulty: "Basic"
---

# What is the difference between HTTP on port 80 and HTTPS on port 443?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ NAT and Port Exhaustion](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/NAT_and_Port_Exhaustion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between HTTP on port 80 and HTTPS on port 443, let's start by breaking down what each protocol does and how they differ.

TL;DR

HTTP (Hypertext Transfer Protocol) operates over port 80 to deliver unencrypted data, while HTTPS (Hypertext Transfer Protocol Secure) uses port 443 to provide encrypted communication. The key differences lie in the security measures, performance implications, and user experience.

#### Triage
What's Being Probed

The question is probing your understanding of web protocols and their characteristics. You need to explain:

What HTTP and HTTPS are.
How they differ in terms of security.
Performance implications.
User experience differences.

#### First-Principles Solution
What HTTP (Port 80) Does

HTTP is a simple, unencrypted protocol designed for transferring data over the web. It operates on port 80 by default and does not encrypt any of the transmitted data. This means that all communication between the client and server can be intercepted if someone has access to the network.

What HTTPS (Port 443) Does

HTTPS, on the other hand, is an extension of HTTP that provides security through encryption. It operates on port 443 by default and uses SSL/TLS protocols to secure data in transit. This ensures that sensitive information like login credentials and financial details are protected from eavesdropping.

Security Differences
HTTP: Data transmitted over HTTP can be intercepted, modified, or replayed.
HTTPS: All data is encrypted using SSL/TLS certificates, making it much harder for attackers to intercept or manipulate the communication.
Performance Implications

While encryption introduces a slight overhead in processing and bandwidth usage, modern web browsers have optimizations that mitigate these impacts. However, the performance difference can be more noticeable on low-bandwidth connections or when dealing with large files.

User Experience Differences
HTTP: No security indicators visible to the user.
HTTPS: Secure site indicators (lock icon) displayed in the browser address bar, providing users with a sense of security.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, HTTP on port 80 is unencrypted and vulnerable to interception, while HTTPS on port 443 provides encryption for secure communication. While there are minor performance considerations, the increased security provided by HTTPS makes it the preferred choice for web applications handling sensitive data.

#### Common mistakes
include:

Not understanding the significance of encryption in securing data.
Ignoring the importance of user experience and security indicators.
Failing to recognize that HTTP and HTTPS have different port numbers (80 vs. 443).

#### Follow-ups
might involve discussing the implementation details of SSL/TLS, how certificate authorities work, or specific use cases where HTTP is still relevant.

**Comparison:**

```
Protocol	Port	Encryption	Security Risk
HTTP	80	No	Yes
HTTPS	443	Yes	No
```

**Code:**

```python
# Example Python code snippet to demonstrate handling a simple HTTP request and an HTTPS request

import requests

def fetch_data_http():
    response = requests.get('http://example.com')
    print(response.text)

def fetch_data_https():
    response = requests.get('https://secure.example.com')
    print(response.text)
```

**Example input / output:**

```text
HTTP Request:
Input: GET http://example.com HTTP/1.1
Output: Response headers and body

HTTPS Request:
Input: GET https://secure.example.com HTTP/1.1
Output: Encrypted response headers and body
```

**Data:**

```json
{
  "HTTP": {
    "port": 80,
    "encryption": false,
    "security_risk": true
  },
  "HTTPS": {
    "port": 443,
    "encryption": true,
    "security_risk": false
  }
}
```

**Flow:**

1. HTTP Request
2. Unencrypted Transmission
3. HTTP Response
4. HTTPS Request
5. Encryption with SSL/TLS
6. Encrypted Transmission
7. Secure HTTPS Response

