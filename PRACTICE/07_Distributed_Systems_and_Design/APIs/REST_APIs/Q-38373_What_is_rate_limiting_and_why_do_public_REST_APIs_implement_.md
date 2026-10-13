---
type: "practice"
difficulty: "Intermediate"
---

# What is rate limiting, and why do public REST APIs implement it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Rate Limiting](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rate limiting is a fundamental technique used to control the number of requests a client can send to a server within a certain time frame. For public REST APIs, implementing rate limiting helps prevent abuse, protect against denial-of-service (DoS) attacks, and ensure fair usage among users. It involves setting constraints on request rates from clients, typically measured in terms of requests per second or per minute.

First, let's break down the question to understand what it is probing:

Definition of Rate Limiting: What does rate limiting entail?
Purpose of Rate Limiting for REST APIs: Why do public REST APIs need this mechanism?

#### Implementation
Details: How can rate limiting be implemented in practice?

Now, let's delve into the core concepts and provide a step-by-step solution from first principles.

Step 1: Understanding Rate Limiting

Rate limiting involves setting limits on how frequently or how many requests can be made by a client to the server. This is often achieved using techniques like fixed-rate limiters (e.g., token bucket) or variable-rate limiters (e.g., leaky bucket).

Token Bucket Algorithm
Token Bucket: A bucket where tokens are added at a steady rate, and clients can "borrow" tokens from the bucket to make requests.
Request Handling:
If there are enough tokens, the client's request is processed and tokens are consumed.
If no tokens are available, the request is rejected.
Leaky Bucket Algorithm
Leaky Bucket: A bucket where a fixed number of tokens are added per unit time, and requests consume one token each. Tokens leak out at a constant rate to simulate old or unused requests.
Request Handling:
If there are enough tokens, the client's request is processed and tokens are consumed.
If no tokens are available, the request is rejected.
Step 2: Why Public REST APIs Need Rate Limiting
Protection Against Abuse: Rate limiting ensures that only legitimate users can make requests, preventing unauthorized access or misuse of the API.
Mitigation of DoS Attacks: By limiting the number of requests per client, public APIs can reduce the impact of DoS attacks, where malicious clients flood the server with numerous requests to overwhelm it.
Fair Usage: Rate limiting helps distribute load more evenly across all users, preventing a few heavy users from monopolizing resources and leaving other users with reduced performance.
Step 3: Implementing Rate Limiting

To implement rate limiting in an HTTP-based REST API, we can use middleware or libraries that enforce these limits. Here’s a simple example using Node.js and Express:

JAVASCRIPT

#### cons
t express = require('express');

#### cons
t rateLimit = require('express-rate-limit');

#### cons
t app = express();

// Configure rate limiter to limit 100 requests per minute from each IP address.

#### cons
t apiLimiter = rateLimit({
    windowMs: 60 * 1000, // 1 minute
    max: 100,
    message: 'Too many API requests, please try again later.'
});

// Apply the rate limiter to all routes
app.use(apiLimiter);

app.get('/api/data', (req, res) => {
    // Simulate fetching data from a database or an external service.
    res.json({ success: true, data: 'Sample Data' });
});

app.listen(3000, () => {

#### cons
ole.log('Server is running on port 3000');
});
Step 4: Common Mistakes and Follow-Ups
Ignoring Edge Cases: Ensure your rate limiter works correctly during sudden bursts of traffic or during server maintenance.
Token Bucket vs. Leaky Bucket: Choose the right algorithm based on your use case. Token bucket is better for managing bursty requests, while leaky bucket is more suitable for sustained load.
Rate Limiting vs. Throttling: Understand the difference; rate limiting can be used to enforce global limits, whereas throttling might focus on specific actions or parameters.

#### Closing Summary

In summary, rate limiting is a crucial technique for public REST APIs, serving to protect against abuse and mitigate DoS attacks while ensuring fair usage among users. Implementing it effectively requires careful consideration of the algorithms used and how they integrate with your existing infrastructure.

**Code:**

```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');

const app = express();

// Configure rate limiter to limit 100 requests per minute from each IP address.
const apiLimiter = rateLimit({
    windowMs: 60 * 1000, // 1 minute
    max: 100,
    message: 'Too many API requests, please try again later.'
});

// Apply the rate limiter to all routes
app.use(apiLimiter);

app.get('/api/data', (req, res) => {
    // Simulate fetching data from a database or an external service.
    res.json({ success: true, data: 'Sample Data' });
});

app.listen(3000, () => {
    console.log('Server is running on port 3000');
});
```

