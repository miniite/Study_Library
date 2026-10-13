---
type: "practice"
difficulty: "Intermediate"
---

# How would you build a gateway that enforces different max-request-body limits per route (1MB for JSON APIs, 500MB for uploads) while streaming so a large upload never buffers fully in gateway memory?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rate Limiting at the Gateway](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Rate_Limiting_at_the_Gateway.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of enforcing different maximum request body limits per route using an API Gateway while stream processing to prevent large uploads from buffering fully, we need to design a solution that intelligently handles incoming requests based on their content type and size. The key idea is to use a combination of HTTP headers for content length checking and streaming APIs to handle the data in chunks.

#### Triage

The question is asking how to implement an API Gateway that can dynamically enforce varying request body limits depending on the route, specifically 1MB for JSON APIs and 500MB for uploads. The interviewer will be probing your understanding of HTTP headers, stream processing, and edge cases such as what happens if a request exceeds the limit.

#### First-Principles Solution

We can achieve this by using an API Gateway framework that supports dynamic routing based on the URL path (route) and content length checking during streaming. The steps are as follows:

Define Routes with Different Limits: Map different routes to their respective maximum request body limits.
Stream Processing: Implement stream processing to handle large uploads without buffering fully in memory.
HTTP Headers for Content Length Checking: Use the Content-Length header to dynamically check and enforce the limit during streaming.
Detailed Implementation

Dynamic Route Mapping:

Map routes to their respective limits using a configuration or an environment variable.

Stream Processing with Node.js (Example): We'll use Node.js for this example due to its event-driven, non-blocking nature which is ideal for stream processing.

JAVASCRIPT

#### cons
t express = require('express');

#### cons
t app = express();

// Configuration for different routes and limits

#### cons
t requestLimits = {
  '/json-endpoint': { limit: '1mb' },
  '/upload-endpoint': { limit: '500mb' }
};

app.use(express.raw({ type: '*/*', limit: '1mb' })); // Default route

function enforceLimit(route, req, res) {

#### cons
t limit = requestLimits[route] ? requestLimits[route].limit : null;
  if (limit && Number(limit.split('b')[0]) * 1024 * 1024 < req.headers['content-length']) {
    return res.status(413).send("Request entity too large");
  }
}

app.use('/json-endpoint', express.raw({ type: 'application/json' }));
app.post('/json-endpoint', (req, res) => {
  enforceLimit('/json-endpoint', req, res);
  // Process JSON request
});

app.post('/upload-endpoint', (req, res) => {
  enforceLimit('/upload-endpoint', req, res);
  // Stream upload handling

#### cons
t chunks = [];
  req.on('data', chunk => {
    if (chunks.reduce((acc, curr) => acc + curr.length, 0) > Number(requestLimits['/upload-endpoint'].limit.split('b')[0]) * 1024 * 1024) {
      return res.status(413).send("Request entity too large");
    }
    chunks.push(chunk);
  });
  req.on('end', () => {

#### cons
t buffer = Buffer.concat(chunks);
    // Handle the complete file

#### cons
ole.log(`Uploaded file of size: ${buffer.length} bytes`);
    res.send('File uploaded successfully');
  });
});

app.listen(3000, () => {

#### cons
ole.log('Server is running on port 3000');
});

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By using dynamic route mapping and stream processing, we can enforce different maximum request body limits per route. The solution involves checking the Content-Length header during streaming to ensure that large uploads do not buffer fully in memory.

#### Common Mistakes
Not handling edge cases where the Content-Length header might be missing or inaccurate.
Failing to stream the data properly, leading to full buffering.
Additional Considerations
Security: Ensure that the limits are enforced securely and cannot be bypassed by malicious clients.
Performance: Optimize the streaming logic to handle large files efficiently without excessive memory usage.
Scalability: Design for horizontal scalability if traffic or file sizes increase significantly.

**Code:**

```javascript
const express = require('express');
const app = express();

// Configuration for different routes and limits
const requestLimits = {
  '/json-endpoint': { limit: '1mb' },
  '/upload-endpoint': { limit: '500mb' }
};

app.use(express.raw({ type: '*/*', limit: '1mb' })); // Default route

function enforceLimit(route, req, res) {
  const limit = requestLimits[route] ? requestLimits[route].limit : null;
  if (limit && Number(limit.split('b')[0]) * 1024 * 1024 < req.headers['content-length']) {
    return res.status(413).send("Request entity too large");
  }
}

app.use('/json-endpoint', express.raw({ type: 'application/json' }));
app.post('/json-endpoint', (req, res) => {
  enforceLimit('/json-endpoint', req, res);
  // Process JSON request
});

app.post('/upload-endpoint', (req, res) => {
  enforceLimit('/upload-endpoint', req, res);
  // Stream upload handling
  const chunks = [];
  req.on('data', chunk => {
    if (chunks.reduce((acc, curr) => acc + curr.length, 0) > Number(requestLimits['/upload-endpoint'].limit.split('b')[0]) * 1024 * 1024) {
      return res.status(413).send("Request entity too large");
    }
    chunks.push(chunk);
  });
  req.on('end', () => {
    const buffer = Buffer.concat(chunks);
    // Handle the complete file
    console.log(`Uploaded file of size: ${buffer.length} bytes`);
    res.send('File uploaded successfully');
  });
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```

**Example input / output:**

```

{ "method": "POST", "url": "/json-endpoint", "headers": { "Content-Length": "950000", "Content-Type": "application/json" }, "body": "{"key": "value"}" }

CODE
Copy

```json
{
  "status": 413,
  "message": "Request entity too large"
}
```

**Flow:**

1. Yes
2. Exceeds Limit
3. Within Limit
4. No
5. Yes
6. Exceeds Limit
7. Within Limit
8. No
9. Receive Request
10. Is Route JSON Endpoint?
11. Check Content-Length
12. Send 413 Response
13. Process JSON Data
14. Is Route Upload Endpoint?
15. Stream Data Handling
16. Send 413 Response
17. Handle Complete File
18. Default Route Handler

