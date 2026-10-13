---
type: "practice"
difficulty: "Basic"
---

# Why should you use HTTPS for REST APIs, and what does it protect against?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Keys vs Tokens](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Keys_vs_Tokens.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a REST API, using HTTPS is crucial because it ensures secure communication between the client and server. HTTPS protects your API from various security threats by encrypting data in transit, verifying the identity of the server, and providing integrity checks to prevent tampering.

Firstly, let's break down what HTTPS does: it establishes an encrypted connection over HTTP (Hypertext Transfer Protocol) using SSL/TLS protocols. This means that any sensitive information, such as user credentials or financial details, are transmitted securely, preventing eavesdropping by attackers. Secondly, HTTPS verifies the server’s identity through certificates issued by trusted Certificate Authorities (CAs), ensuring that you’re communicating with the intended service and not a malicious one attempting to intercept your data.

To understand why this is important, let's consider some common security threats:

Man-in-the-Middle Attacks: Without encryption, an attacker can sit between the client and server and intercept or modify data being exchanged.
Data Leakage: Sensitive information like passwords or credit card numbers could be seen by unauthorized parties if they manage to breach the network.
Repudiation: HTTPS provides a way to verify that messages are not tampered with in transit, which is essential for maintaining trust and accountability.

Now, let's walk through how HTTPS protects against these threats:

Encryption: SSL/TLS encrypts data being sent between the client and server. Even if an attacker manages to intercept the data, they cannot read it without the decryption key.
Server Authentication: When a client connects to a secure server, the server presents a digital certificate that can be checked against a trusted CA’s list of valid certificates. This ensures that you are communicating with the correct server and not a malicious one.

Here's an example scenario: Imagine a user logs into your application using their credentials. If HTTPS is used, these credentials are encrypted during transmission, making it significantly harder for anyone to intercept them even if they are snooping on the network traffic.

In summary, implementing HTTPS in REST APIs ensures that:

Data transmitted between client and server remains private.
The authenticity of the server can be verified.
Messages cannot be altered or tampered with during transit.

**Code:**

```text
// Example code snippet for setting up a simple HTTPS server using Node.js
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('/path/to/privkey.pem'),
  cert: fs.readFileSync('/path/to/cert.pem')
};

https.createServer(options, (req, res) => {
  res.writeHead(200);
  res.end('Hello World\n');
}).listen(8443);

console.log('HTTPS server is running on port 8443');
```

