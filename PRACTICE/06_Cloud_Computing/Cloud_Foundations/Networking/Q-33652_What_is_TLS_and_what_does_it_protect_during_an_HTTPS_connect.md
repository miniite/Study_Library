---
type: "practice"
difficulty: "Basic"
---

# What is TLS and what does it protect during an HTTPS connection?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ mTLS and Zero Trust](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/mTLS_and_Zero_Trust.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TLS, or Transport Layer Security, is a protocol that provides secure communication over computer networks. During an HTTPS connection, TLS encrypts the data being transmitted between the client (e.g., a web browser) and the server. This ensures that sensitive information like passwords and credit card details remain confidential and prevent eavesdropping.

TLS operates at the Transport Layer of the OSI model, which is why it's part of HTTP to form HTTPS. It works by establishing an encrypted channel between the two parties using symmetric key cryptography for data encryption and public key cryptography (asymmetric) for key exchange. The connection starts with a handshake process where both sides agree on a set of security parameters.

TLS protects several aspects during an HTTPS connection:

Confidentiality: Ensures that data cannot be read by unauthorized entities.
Integrity: Guarantees that the data is not altered in transit without detection.
Authentication: Verifies the identity of both parties involved in the communication to prevent man-in-the-middle attacks.

TLS uses a combination of SSL (Secure Sockets Layer) and various versions thereof, with TLS 1.3 being the latest and most secure version. It relies on certificates issued by trusted Certificate Authorities (CAs) to verify the server’s identity.

Now, let's dive into the technical details from first principles.

**Comparison:**

|TLS Handshake Steps|
|---|
|1. Client Hello - Initial message from client to server containing supported protocols, cipher suites, etc.|
|2. Server Hello - Server responds with selected protocol version and cipher suite.|
|3. Certificate - Server sends its certificate for verification.|
|4. Client Key Exchange - Client generates a session key using the agreed-upon algorithm.|
|5. Change Cipher Spec - Signals that all following messages will be encrypted.|
|6. Finished - Final message from each side to confirm secure connection established.|

**Code:**

```
MERMAID
Copy
sequenceDiagram
    participant Client
    participant Server
    Client->>Server: TLS Handshake
    alt Successful handshake
        Server-->>Client: Server Hello
        Client-->>Server: Client Hello
        Server-->>Client: Certificate
        Client-->>Server: Client Key Exchange, Change Cipher Spec
        Server-->>Client: Finished
        Client-->>Server: Finished
    end
    Server->>Client: Encrypted Application Data
    Client->>Server: Encrypted Application Data
```

**Example input / output:**

```

Here is a hypothetical scenario of an HTTPS request and response with TLS encryption:

Request from Client to Server

HTTP
Copy
GET / HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.128 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Connection: close

Response from Server to Client

HTTP
Copy
HTTP/1.1 200 OK
Date: Mon, 01 Jan 2023 12:00:00 GMT
Server: Apache/2.4.46 (Ubuntu)
Content-Type: text/html; charset=UTF-8
Content-Length: 144

<html>
    <head><title>Example Domain</title></head>
    <body>
        <h1>Example Domain</h1>
        <p>This domain is for use in illustrative examples in documents. You may use this
        domain in literature without prior coordination or asking for permission.</p>
    </body>
</html>

The data above is encrypted during the TLS handshake and transmitted securely.
```

**Data:**

```json
{
  "TLSVersion": "1.3",
  "CipherSuite": "TLS_AES_256_GCM_SHA384",
  "KeyExchangeAlgorithm": "Elliptic Curve Diffie-Hellman (ECDHE)",
  "CertificateChain": [
    {
      "Subject": "example.com",
      "Issuer": "Let's Encrypt Authority X1"
    }
  ]
}
```

**Flow:**

1. Client
2. Server
3. TLS Handshake
4. Encryption setup
5. Data exchange (Encrypted)
6. Decryption on receiving side
7. Connection closed

