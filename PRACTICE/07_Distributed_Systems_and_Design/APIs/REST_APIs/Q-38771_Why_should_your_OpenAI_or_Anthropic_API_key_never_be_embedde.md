---
type: "practice"
difficulty: "Intermediate"
---

# Why should your OpenAI or Anthropic API key never be embedded directly in frontend JavaScript, and what is the correct REST architecture instead?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of URLs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Keeping_Secrets_out_of_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When developing web applications that interact with AI APIs like OpenAI or Anthropic, embedding your API keys directly into frontend JavaScript is a significant security risk. This approach exposes your sensitive credentials to potential vulnerabilities and can lead to unauthorized access to your API resources. Instead, the correct REST architecture involves using server-side handling of API requests, which provides better security and flexibility.

To understand why this matters, let's break down the key aspects:

Security Risks: Embedding API keys in frontend JavaScript exposes them to clients who can inspect their code or use tools like browser developer consoles to extract these secrets.
Client-Side vs Server-Side Handling: By keeping API keys on the server side and using client-side JavaScript only for requesting data from the server, you prevent direct access to sensitive information.

Now, let's explore a step-by-step solution:

Step 1: Understand the Problem

The problem is that exposing API keys in frontend code can lead to unauthorized access. To solve this, we need to ensure that API requests are made through secure channels managed by the server.

Step 2: Server-Side Authentication and Authorization

Implement a server-side mechanism for authentication and authorization. This can be done using OAuth, JWT (JSON Web Tokens), or other secure methods. Here’s an example of how it works:

User Login: Users authenticate via a web form that sends their credentials to the backend.
Generate Token: Upon successful login, a token is generated and returned to the client. This token contains necessary claims about the user's identity and permissions.
Token Storage: The frontend stores this token securely (e.g., in localStorage or as an HTTP-only cookie).
API Requests: When making API requests, the frontend sends this token along with the request.
Step 3: Example Implementation

Here’s a simplified example using Express.js and JWT:

JAVASCRIPT
// server.js

#### cons
t express = require('express');

#### cons
t jwt = require('jsonwebtoken');

#### cons
t app = express();
app.use(express.json());

#### cons
t SECRET_KEY = 'your-secret-key';

// User authentication endpoint
app.post('/login', (req, res) => {

#### cons
t { username, password } = req.body;
    if (username === 'admin' && password === 'password') {

#### cons
t token = jwt.sign({ username }, SECRET_KEY);
        res.json({ token });
    } else {
        res.status(401).send('Invalid credentials');
    }
});

// Protected API endpoint
app.get('/api/data', verifyToken, (req, res) => {
    res.json({ data: 'some sensitive information' });
});

function verifyToken(req, res, next) {

#### cons
t token = req.header('Authorization').replace('Bearer ', '');
    if (!token) return res.status(401).send('Access denied');

    try {

#### cons
t decoded = jwt.verify(token, SECRET_KEY);
        req.user = decoded;
        next();
    } catch (err) {
        res.status(400).send('Invalid token');
    }
}

app.listen(3000, () => console.log('Server running on port 3000'));
Step 4: Frontend Integration

On the frontend, you would include an Authorization header with the JWT when making API requests:

JAVASCRIPT
// client.js
fetch('/api/data', {
    method: 'GET',
    headers: {
        'Authorization': 'Bearer ' + localStorage.getItem('token'),
    }
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));
Step 5: Closing Summary and Common Mistakes

By keeping your API keys on the server side, you significantly enhance the security of your application. Common mistakes include:

Storing Tokens Insecurely: Ensure tokens are stored securely (e.g., not in plain text).
Using Weak Passwords: Implement strong password policies for user authentication.
Exposing Server-Side Logic: Avoid exposing server-side logic that could be exploited by attackers.
Final Thoughts

Always prioritize security in your application development. By using proper REST architecture and secure practices, you can protect sensitive API keys and ensure the integrity of your application.

**Code:**

```

// server.js example const express = require('express'); const jwt = require('jsonwebtoken');

const app = express(); app.use(express.json());

const SECRET_KEY = 'your-secret-key';

app.post('/login', (req, res) => { const { username, password } = req.body; if (username === 'admin' && password === 'password') { const token = jwt.sign({ username }, SECRET_KEY); res.json({ token }); } else { res.status(401).send('Invalid credentials'); } });

app.get('/api/data', verifyToken, (req, res) => { res.json({ data: 'some sensitive information' }); });

function verifyToken(req, res, next) { const token = req.header('Authorization').replace('Bearer ', ''); if (!token) return res.status(401).send('Access denied');

CODE
Copy
try {
    const decoded = jwt.verify(token, SECRET_KEY);
    req.user = decoded;
    next();
} catch (err) {
    res.status(400).send('Invalid token');
}

}

app.listen(3000, () => console.log('Server running on port 3000'));

CODE
Copy

```javascript
// client.js example
fetch('/api/data', {
    method: 'GET',
    headers: {
        'Authorization': 'Bearer ' + localStorage.getItem('token'),
    }
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));
```

