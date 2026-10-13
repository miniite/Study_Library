---
type: "practice"
difficulty: "Deep"
---

# How would you design a REST API so that an internal service and an external partner hit the 'same' endpoint but the partner gets stricter validation, rate limits, and field redaction, without duplicating the handler?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Authorization Design](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Authorization_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a REST API where both an internal service and an external partner can hit the same endpoint while ensuring that the partner receives additional validations, rate limits, and field redactions, we need to implement a layered approach that allows for flexible but controlled access based on the client's identity. This solution ensures that we do not duplicate code or logic unnecessarily.

Firstly, let’s understand what the problem entails: We have an endpoint /api/data which both internal services and external partners can use. However, the validation rules, rate limits, and redaction policies should differ between these two groups. Internal services might require minimal validation and no restrictions, whereas external partners need stricter controls to protect sensitive data.

#### Triaging the Question

The core of this question is about designing an API that allows for different levels of access control without duplicating code. This involves:

Differentiating between internal and external clients.
Applying appropriate policies based on client type.
Managing shared resources efficiently while ensuring security and performance.

#### First-Principles Solution
Authentication and Authorization: Use a robust authentication mechanism to identify the client type (internal or external).
Client-Specific Middleware: Implement middleware that applies specific policies based on the authenticated client.
Dynamic Validation Logic: Create flexible validation logic that can be adjusted depending on the client’s role.
Rate Limiting and Throttling: Configure rate limits differently for internal services compared to external partners.
Field Redaction: Implement a mechanism to selectively redact fields based on the client type.
Detailed Steps

Authentication:

Use OAuth2 or JWT for authentication. For example, internal services might use a private key, while external partners could use public keys or tokens issued by us.

Client-Specific Middleware:

Create middleware functions that can be dynamically applied to the endpoint based on the client’s identity.

Dynamic Validation Logic:

Write validation logic in a way that it checks specific fields only if certain conditions are met (e.g., external == true).

Rate Limiting and Throttling:

Set up rate limiting policies using tools like Redis or Envoy. Define different limiters for internal services and external partners.

Field Redaction:

Implement a mechanism to selectively hide fields based on the client’s role. For example, certain fields might only be visible to internal services but hidden from external partners.

#### Example
Implementation

Here is an example implementation using Node.js with Express:

JAVASCRIPT

#### cons
t express = require('express');

#### cons
t jwt = require('jsonwebtoken');

#### cons
t rateLimit = require('express-rate-limit');

#### cons
t app = express();
app.use(express.json());

// Sample JWT secret for internal services

#### cons
t INTERNAL_SECRET = 'internal_secret_key';
// Public key or token for external partners

#### cons
t EXTERNAL_TOKEN = 'external_partner_token';

// Middleware to authenticate and apply policies
function authenticateClient(req, res, next) {
    if (req.headers['x-api-key'] === EXTERNAL_TOKEN) {
        req.clientType = 'external';
        // Apply stricter validation and rate limits for external clients
    } else {
        req.clientType = 'internal';
        // Minimal validation and no rate limiting for internal services
    }
    next();
}

// Validation logic based on client type
function validateData(req, res) {
    if (req.clientType === 'external') {
        // Stricter validation rules here
    } else {
        // Minimal validation here
    }
}

// Rate limit middleware

#### cons
t limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // maximum calls per windowMs
});

app.use('/api/data', authenticateClient);
app.post('/api/data', limiter, validateData, (req, res) => {
    if (req.clientType === 'external') {

#### cons
t redactedData = req.body;
        // Redact sensitive fields
        delete redactedData.sensitiveInfo;
        res.json(redactedData);
    } else {
        res.json(req.body); // Full data for internal services
    }
});

app.listen(3000, () => {

#### cons
ole.log('Server is running on port 3000');
});

#### Closing Summary

By using a layered approach with middleware and dynamic validation logic, we can design an API that meets the requirements without duplicating code or handlers. This solution ensures flexible access control while maintaining security and performance.

#### Common Mistakes/Follow-Ups

Mistake: Failing to differentiate between internal and external clients properly.

#### Follow-Up
Always ensure robust authentication mechanisms are in place and client identification is clear.

Mistake: Over-complicating validation logic, leading to unnecessary complexity.

#### Follow-Up
Keep the validation logic simple yet effective. Use environment-specific configurations if needed.

**Code:**

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const rateLimit = require('express-rate-limit');

const app = express();
app.use(express.json());

function authenticateClient(req, res, next) {
    if (req.headers['x-api-key'] === EXTERNAL_TOKEN) {
        req.clientType = 'external';
        // Apply stricter validation and rate limits for external clients
    } else {
        req.clientType = 'internal';
        // Minimal validation and no rate limiting for internal services
    }
    next();
}

function validateData(req, res) {
    if (req.clientType === 'external') {
        // Stricter validation rules here
    } else {
        // Minimal validation here
    }
}

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // maximum calls per windowMs
});

app.use('/api/data', authenticateClient);
app.post('/api/data', limiter, validateData, (req, res) => {
    if (req.clientType === 'external') {
        const redactedData = req.body;
        // Redact sensitive fields
        delete redactedData.sensitiveInfo;
        res.json(redactedData);
    } else {
        res.json(req.body); // Full data for internal services
    }
});

app.listen(3000, () => {
    console.log('Server is running on port 3000');
});
```

**Flow:**

1. Internal
2. External
3. Yes
4. No
5. Client Request
6. Authenticate Client
7. Internal or External?
8. Minimal Validation and NoRate Limiting
9. Apply Redaction Logic?
10. Redact Sensitive Fields
11. Strict Validation, Apply RateLimits
12. Respond with Redacted Data
13. Respond with Full Data
14. Send Request to Handler

