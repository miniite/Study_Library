---
type: "practice"
difficulty: "Intermediate"
---

# A client sends an Accept-Encoding of br but one upstream only supports gzip; how does the gateway negotiate, re-encode, or pass through compression, and what are the CPU vs bandwidth trade-offs of re-compressing at the edge?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Response Transformation and Upstream Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: In this scenario, an API Gateway needs to handle a client request with Accept-Encoding: br while ensuring it works with an upstream service that only supports gzip. The solution involves negotiation and re-encoding or passing through the compression based on performance trade-offs between CPU usage and bandwidth savings.

#### Triage
The question is about how an API Gateway should manage requests with different encoding preferences when the backend service does not support all client preferences. Specifically, it probes our understanding of HTTP negotiation mechanisms, re-encoding processes, and the balance between CPU load and network efficiency.

#### First Principles Solution

Client Request Handling: The client sends a request with Accept-Encoding: br, indicating that it prefers to receive compressed responses using Brotli.
Upstream Service Support: The upstream service only supports gzip encoding, meaning the gateway needs to either re-compress or pass through gzip data depending on efficiency and constraints.
HTTP Negotiation: Use HTTP content negotiation headers like Accept-Encoding. If the client supports both encodings, use a more efficient one; if not, fall back to a supported option.
Re-encoding Strategy: Re-compressing at the gateway can save bandwidth but will increase CPU usage and latency. Passing through gzip might be less efficient in terms of bandwidth but reduces computational overhead.
Performance Trade-offs: Evaluate based on the specific use case, such as the expected payload size, client proximity to the gateway, and network conditions.

#### Implementation
Steps:

Check Client Preferences: Examine the incoming request's Accept-Encoding header for supported encodings (e.g., br, gzip).
Evaluate Upstream Support: Determine if the upstream service can handle both requested encodings or only one.
Negotiate Compression: If the client supports multiple encodings and the gateway can re-compress, negotiate using Content-Encoding headers.
Re-encode Data: Re-compress data from gzip to br if possible.
Pass Through: Directly pass through gzip content if re-encoding is not feasible or less efficient.

#### Closing Summary
and Common Mistakes:

The key idea here is to efficiently manage client requests and upstream service capabilities while balancing CPU and bandwidth constraints.

#### Common mistakes
include over-relying on re-encoding, failing to consider the trade-offs between computational efficiency and network performance, or ignoring the actual implementation details such as header handling.

**Code:**

```python
# Example Python code snippet for handling request headers and response encoding

from flask import Flask, request, make_response
import gzip
import brotli

app = Flask(__name__)

@app.route('/api')
def api_gateway():
    # Check client's Accept-Encoding header
    accept_encoding = request.headers.get('Accept-Encoding', '')

    # Example upstream service response (gzipped)
    upstream_data = b'Example data that has been compressed using gzip'

    if 'br' in accept_encoding:  # Client prefers Brotli, but gateway supports Gzip only
        # Re-encode from gzip to br (if possible)
        compressed_data = brotli.compress(upstream_data)
        response = make_response(compressed_data)
        response.headers['Content-Encoding'] = 'br'
    else:
        # Pass through the gzip content directly if client doesn't support Brotli
        compressed_data = upstream_data
        response = make_response(compressed_data)
        response.headers['Content-Encoding'] = 'gzip'

    return response

# Note: This is a simplified example and may require additional error handling and configuration.
```

**Example input / output:**

```

Input Request Headers:

HTTP
Copy
GET /api HTTP/1.1
Host: example.com
Accept-Encoding: br

Output Response Headers and Content (after re-encoding):

HTTP
Copy
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: [length of compressed content]
Content-Encoding: br

[Compressed data using Brotli algorithm]
```

**Data:**

```json
{
  "request_headers": {
    "Accept-Encoding": "br"
  },
  "upstream_data": {
    "compressed_with_gzip": true,
    "content": "Example data that has been compressed using gzip"
  }
}
```

