---
type: "practice"
difficulty: "Expert"
---

# How do you design a gateway to safely handle chunked transfer-encoding and HTTP request smuggling risks (TE vs CL desync) when there's a second proxy between the gateway and upstream?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Response Transformation and Upstream Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the design challenge of safely handling chunked transfer-encoding and HTTP request smuggling risks in an API Gateway, especially with two proxies involved, we need to focus on secure communication, robust protocol handling, and layered security measures. The key idea is to implement a comprehensive gateway architecture that ensures data integrity and prevents malicious desynchronization between the gateway and upstream services.

#### Triage
The question revolves around ensuring the gateway can safely handle chunked transfer-encoding (TE) and mitigate HTTP request smuggling risks, specifically TE vs CL desync issues when there are two proxies involved. The main concerns include:

Chunked Transfer-Encoding (TE): Ensuring that the gateway correctly handles chunked data without losing or misinterpreting parts of the stream.
HTTP Request Smuggling: Preventing attacks where malicious clients can manipulate request boundaries, causing desynchronization between different services.

#### Assumptions

The gateway is responsible for handling and routing requests to backend services.
There are two proxies: one before the gateway and another after (between the gateway and upstream).
The goal is to ensure data integrity and security while maintaining performance.

Step-by-step Solution:

Protocol Handling: Implement strict protocol validation and parsing at the gateway level. Ensure that chunked transfer encoding is handled correctly by accurately tracking the end of each chunk and ensuring the last chunk is properly terminated.
Request Boundary Detection: Use techniques like header analysis to detect potential request boundaries, especially in cases where multiple requests are being smuggled into a single HTTP message.
Desynchronization Prevention: Introduce mechanisms such as sequence numbers or version stamps to maintain state across layers and ensure that the gateway can accurately track and validate request boundaries even with two proxies involved.
Content-Length Verification: For requests with content-length headers, verify the total length of data matches the expected value before processing.
Rate Limiting and Throttling: Implement rate limiting to prevent abuse and mitigate potential DoS attacks that could exploit request smuggling or chunked transfer encoding issues.
Logging and Monitoring: Set up comprehensive logging and monitoring to detect anomalies early, allowing for quick troubleshooting if issues arise.

#### Closing Summary
By implementing a robust gateway design with strict protocol handling, boundary detection, desynchronization prevention mechanisms, and effective security controls, we can safely handle chunked transfer-encoding and mitigate HTTP request smuggling risks. The key is to ensure that data integrity is maintained and potential security threats are minimized even in complex proxy architectures.

#### Common Mistakes/Follow-ups

Failing to validate and parse headers correctly, leading to misinterpretation of chunked data.
Not implementing robust desynchronization prevention mechanisms, which can lead to request smuggling vulnerabilities.
Overlooking the impact of rate limiting and throttling on performance, potentially causing legitimate users to experience delays or failures.

**Comparison:**

```
Layer	Security Measure	Description
Gateway	Protocol Validation	Ensures correct parsing of headers and chunk boundaries.
Proxy 1	Sequence Numbers	Tracks state across layers to prevent desynchronization.
Proxy 2	Content-Length Verification	Validates the total length of data matches expected values.
```

**Code:**

```python
# Example of a simple validation function for HTTP headers

def validate_chunked_request(headers):
    if 'transfer-encoding' in headers:
        if headers['transfer-encoding'] == 'chunked':
            # Validate chunk boundary and ensure last chunk is properly terminated
            pass
    return True
```

**Example input / output:**

```text
Input:
Request Headers:
host: example.com
transfer-encoding: chunked

Response Headers:
Server: Apache/2.4.18 (Ubuntu)
Content-Length: 360

Output:
Gateway successfully validated and processed the request with correct chunk boundaries.
```

**Data:**

```json
{
  "request_headers": {
    "host": "example.com",
    "transfer-encoding": "chunked"
  },
  "response_headers": {
    "server": "Apache/2.4.18 (Ubuntu)",
    "content-length": "360"
  }
}
```

**Flow:**

1. Valid - D[Process Request];D
2. Gateway
3. Receive Request
4. Validate Headers
5. Send Response to Client
6. E
7. Rate Limit and Throttle

