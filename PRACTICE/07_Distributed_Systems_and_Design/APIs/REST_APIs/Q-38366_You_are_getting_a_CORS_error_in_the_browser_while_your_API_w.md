---
type: "practice"
difficulty: "Deep"
---

# You are getting a CORS error in the browser while your API works fine in Postman. How would you debug and fix this?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CORS](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When encountering a CORS (Cross-Origin Resource Sharing) error in a browser, it usually means that the server is not properly configured to handle requests from your client-side application's origin. This error can be frustrating because while Postman works fine, browsers enforce stricter security policies. Here’s how you would debug and fix this issue:

Understand the Error: CORS errors typically look like Access to XMLHttpRequest at '...' from origin '...' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. This means that the server did not include an appropriate Access-Control-Allow-Origin header in its response.

Restate the Question and Assumptions: The question requires understanding the CORS policy, which involves ensuring the correct headers are set when making requests from one origin to another. We need to identify whether this is a client-side issue or a server-side configuration problem.

Step-by-Step Solution:

Client-Side Investigation: First, ensure that your client application is sending the request correctly and that there are no issues with the browser settings.
JAVASCRIPT
// Example of a fetch request in JavaScript
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));
Server-Side Configuration: Investigate the server to see if it is correctly setting up CORS. This involves configuring your backend to include the necessary headers in the responses.
JAVASCRIPT
// Example of a Node.js Express configuration for CORS

#### cons
t express = require('express');

#### cons
t cors = require('cors');

#### cons
t app = express();

// Middleware setup for allowing specific origins
app.use(cors({
  origin: 'http://example.com',  // Specify the allowed origin(s)
  methods: ['GET', 'POST'],      // Specify allowed HTTP methods
  credentials: true              // Allow cookies and other credentials if needed
}));

app.get('/data', (req, res) => {
  res.json({ message: "Hello World" });
});

app.listen(3000, () => console.log('Server is running on port 3000'));

Testing and Validation: Once the server-side changes are made, you should test again using both Postman and the browser to ensure that the CORS issues are resolved.

#### Closing Summary
and Common Mistakes/Follow-Ups:

Summary: Properly configuring CORS involves setting up the correct headers on your server to allow requests from specific origins.

#### Common Mistakes
Forgetting to add credentials: true if cookies or other credentials are needed in the request.
Misconfiguring the allowed methods and origins, leading to partial functionality.
Not testing thoroughly across different browsers and environments.

**Code:**

```javascript
// Example of a Node.js Express configuration for CORS
const express = require('express');
const cors = require('cors');

const app = express();

app.use(cors({
  origin: 'http://example.com',
  methods: ['GET', 'POST'],
  credentials: true
}));

app.get('/data', (req, res) => {
  res.json({ message: "Hello World" });
});

app.listen(3000, () => console.log('Server is running on port 3000'));
```

