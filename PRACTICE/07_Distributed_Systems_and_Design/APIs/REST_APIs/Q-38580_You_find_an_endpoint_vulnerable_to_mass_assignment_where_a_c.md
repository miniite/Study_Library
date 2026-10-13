---
type: "practice"
difficulty: "Deep"
---

# You find an endpoint vulnerable to mass assignment where a client set role=admin. Beyond blocking that field, how do you systematically prevent this class of bug across all write endpoints?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Authorization Design](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Authorization_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To systematically prevent the class of bugs related to mass assignment in REST API endpoints, especially when dealing with sensitive fields like role, we need a comprehensive approach that includes both technical and design improvements. The core idea is to ensure that any client-provided data is validated and sanitized before being stored or used in the application. This involves several steps: input validation, parameterized queries, access control checks, and secure coding practices.

#### Triaging the Question

The question asks for a systematic method to prevent mass assignment vulnerabilities across all write endpoints. The interviewer is probing your understanding of security best practices, validation techniques, and how to implement these in a scalable manner. Assumptions include that we are dealing with an application using REST APIs where user-provided data can be directly manipulated by the backend.

#### Solution

Input Validation:

Implement strict input validation for each field in your write endpoints. This involves checking types, formats, and allowed values.
For example, if role is a string, ensure it only accepts certain predefined roles like 'user', 'admin', etc., using regular expressions or a whitelist approach.

Parameterized Queries:

Use parameterized queries to protect against SQL injection and other injection attacks. This ensures that user input is treated as data rather than executable code.
For example, in a database query like SELECT * FROM users WHERE role = ?, use a placeholder for the role value.

Access Control Checks:

Ensure that the role or any sensitive field can only be modified by authorized users with appropriate permissions. This involves implementing proper authentication and authorization mechanisms.
For instance, an admin user might have the permission to set another user's role, but a regular user should not have this capability.

Secure Coding Practices:

Follow secure coding guidelines such as OWASP’s Secure Coding Practices to avoid common security pitfalls.
Regularly review and refactor code to ensure all write operations are safe and validated.

Automated Testing:

Implement automated tests to catch these kinds of vulnerabilities early in the development process. Tools like static analysis tools can help identify potential issues.
For example, use unit tests that mock user input and verify that validation logic is correctly applied.

Logging and Monitoring:

Log suspicious activities or attempted mass assignments so you can detect and respond to security incidents quickly.
Monitor logs for unusual patterns of requests that might indicate a vulnerability being exploited.

#### Closing Summary

By systematically addressing the root causes through input validation, parameterized queries, access control checks, secure coding practices, automated testing, and logging, we can significantly reduce the risk of mass assignment vulnerabilities in REST APIs. It's important to adopt these practices across all write endpoints to maintain robust security posture.

#### Common Mistakes/Follow-Ups
Failing to Validate Input: Always validate user input at multiple levels (client-side and server-side).
Over-reliance on Whitelisting: While whitelisting is good, it’s equally important to implement blacklisting and other checks.
Ignoring Automated Testing: Regular code reviews and automated testing can help catch these issues early.
Triaging the Follow-Up

If asked about implementing these practices in a specific technology stack (e.g., Node.js with Express), you could discuss how middleware like Helmet for security headers, or libraries like Joi for validation can be used. If the question extends to broader architectural decisions, you might explore microservices architecture and API gateways that enforce security policies.

**Comparison:**

```
Step	Action
1	Implement input validation
2	Use parameterized queries
3	Apply access control
4	Follow secure coding practices
5	Integrate automated testing
6	Enable logging and monitoring
```

**Code:**

```javascript
// Example of input validation in Node.js using Joi
const Joi = require('joi');

function validateUser(user) {
  const schema = Joi.object({
    role: Joi.string().valid('user', 'admin').required(),
    // other fields...
  });

  return schema.validate(user);
}
```

**Example input / output:**

```text
// Mock input
Input: {
  "role": "admin",
  // other fields...
}

// Validated Output (if role is invalid)
Error: "Role must be one of ['user', 'admin']"
```

**Data:**

```json
{
  "role": "admin",
  "username": "test_user"
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Validation Middleware
5. Valid Data?
6. Process Request
7. Reject or Log Error

