---
type: "practice"
difficulty: "Expert"
---

# How do you design a gateway to enforce request signing where the signature covers method, path, body hash, and headers, and how does the gateway verify body hash while streaming a large body it hasn't fully read?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Security and Data Masking](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Security_and_Data_Masking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Designing an API Gateway to enforce request signing involves securing each part of the request (method, path, headers, and body). The gateway needs to ensure that the client-side has a way to generate the signature based on these components without needing to fully read the body. This can be achieved using HMAC or similar cryptographic techniques, where the body hash is computed separately and included in the signing process.

#### Triage
The question asks for designing an API Gateway that enforces request signing with specific criteria (method, path, headers, and body hash). We need to ensure that the gateway can verify these components without needing to fully read the body. This involves understanding how to securely compute and validate hashes in a streaming context.

#### First-principles Solution

HMAC for Security: Use HMAC (Hash-based Message Authentication Code) to create a signature over the request data.
Streaming Body Hashing: Compute the hash of the body while it's being streamed, ensuring that no full read is required until validation.
Gateway Architecture: Design the gateway to handle streaming and validate the signature in parallel.

Step-by-step Explanation:

HMAC Implementation:

The client will compute an HMAC over the request components (method, path, headers, body hash) using a secret key shared between the client and server.
For example, if the method is POST, path is /api/v1/resource, headers include Content-Type, and the body has been hashed to body_hash, the HMAC would be computed over these parts.

Streaming Hash Computation:

The gateway should support streaming of large bodies. As data arrives from the client, it can start computing a hash (e.g., SHA-256) without needing to buffer the entire body.
Use a library like crypto in Node.js or hashlib in Python to compute the hash incrementally.

Gateway Design:

The gateway should be designed to handle incoming requests in streaming mode, ensuring it can process parts of the request as they arrive.
Validate the signature as soon as all necessary parts (method, path, headers) are available and the body hash is computed.

Parallel Validation:

Once the entire request has been received, validate the HMAC against the combined method, path, headers, and body hash.
If validation passes, process the request; otherwise, reject it immediately.

#### Closing Summary
+ Common Mistakes/Follow-ups:

Ensure the gateway supports streaming to avoid memory issues with large bodies.
Incorrectly buffering the entire body can lead to performance degradation or out-of-memory errors.
Failure modes include invalid signatures due to timing issues in computing hashes or HMACs.

#### Follow-up
questions might involve discussing how to handle different types of attacks (e.g., replay attacks) and implementing additional security measures like rate limiting.

**Comparison:**

```
Component	Description
Method	HTTP method of the request (e.g., GET, POST)
Path	The URL path where the API is located
Headers	Request headers that need to be included in the HMAC
Body Hash	Hash of the body content, computed incrementally
```

**Code:**

```python
import hashlib

def compute_body_hash(body):
    return hashlib.sha256(body).hexdigest()

def generate_signature(method, path, headers, body_hash, secret_key):
    message = f"{method}{path}{body_hash}{headers}"
    return hmac.new(secret_key.encode(), msg=message.encode(), digestmod=hashlib.sha256).hexdigest()
```

**Example input / output:**

```

Input:

Method: POST
Path: /api/v1/resource
Headers: {"Content-Type": "application/json"}
Body (streamed incrementally): {"key": "value"}

Output:

Generated signature: 47e985d6b03f21c4

Validation:

Gateway receives the signature and validates it against the method, path, headers, and body hash.
BASH
Copy
# Example validation in shell
signature="47e985d6b03f21c4"
method="POST"
path="/api/v1/resource"
headers="Content-Type: application/json"
body_hash="9e107d9d372bb6826bd81d3542a419d6"

# Compute expected signature
expected_signature=$(generate_signature "$method" "$path" "$headers" "$body_hash")

if [ "$signature" == "$expected_signature" ]; then
    echo "Signature valid"
else
    echo "Invalid signature"
fi
```

**Data:**

```json
{
  "method": "POST",
  "path": "/api/v1/resource",
  "headers": {"Content-Type": "application/json"},
  "body_hash": "9e107d9d372bb6826bd81d3542a419d6"
}
```

**Flow:**

1. Yes
2. Yes
3. Valid
4. Invalid
5. Receive request
6. Is method, path, and headersavailable?
7. Compute body hash whilestreaming
8. Are all components ready forsignature generation?
9. Generate HMAC using allcomponents
10. Validate HMAC againstreceived signature
11. Proceed with requestprocessing
12. Reject request

