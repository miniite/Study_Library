---
type: "practice"
difficulty: "Deep"
---

# How do environment variables differ between build time and run time for a frontend app?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Artifact Promotion Across Environments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Artifact_Promotion_Across_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Environment variables in a frontend application can be managed differently during build time versus runtime, with significant implications on configuration flexibility, security, and deployment processes. At build time, we often use environment-specific configurations to customize the application's behavior before packaging it for deployment. At runtime, these same or different environment variables are used to dynamically configure the app based on the current environment context.

#### Triage

The question is asking about how environment variables are handled differently at two distinct phases of a frontend application’s lifecycle: build time and run time. The interviewer likely wants to understand your knowledge of:

Build Time: How environment variables are processed, possibly through configuration files or scripts.
Run Time: How these variables are accessed by the application during its execution.

#### First Principles Solution
Build Time

At build time, the primary goal is to customize and optimize the application for specific environments (development, testing, production). Here’s how environment variables can be managed:

Configuration Files: Often stored in .env files or similar formats.
Build Scripts: Use tools like dotenv, npm env, or custom scripts that parse these files to inject values into the build process.
Environment-Specific Configurations: Different .env files for each environment (e.g., .env.development, .env.production).
CI/CD Integration: Scripts in CI pipelines, such as npm run build --env=production, can ensure that specific configurations are applied correctly.

#### Example
of a .env.development file:

PLAINTEXT
REACT_APP_API_URL=http://localhost:8080/api
Run Time

At runtime, the application needs to access these environment variables to configure its behavior dynamically. Here’s how this is typically handled:

.env Files: These files are loaded during development and should be excluded from source control.
Environment Variables: Set through system configuration or deployment tools like Docker, Kubernetes, or cloud providers (e.g., AWS).
Accessing Variables: Using environment variables in code, typically via process.env in Node.js, or equivalent mechanisms in other languages.

#### Example
of accessing an environment variable:

JAVASCRIPT

#### cons
t apiUrl = process.env.REACT_APP_API_URL;
Key Considerations:
Security: Avoid hardcoding sensitive information like API keys directly in source files.
Flexibility: Support for different environments (development, staging, production).

#### Cons
istency: Ensure that the build and runtime configurations are consistent.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, managing environment variables at build time and run time involves different approaches to ensure the application is configured correctly and securely. Key mistakes include:

Hardcoding sensitive information.
Not properly excluding .env files from source control.
Ignoring differences between development and production environments.

For follow-ups, an interviewer might ask about specific tools or best practices for handling environment variables, such as secrets management, CI/CD integration, or dynamic configuration during runtime. These discussions can delve into the nuances of security, performance, and scalability.

**Code:**

```javascript
// Example .env.development file
REACT_APP_API_URL=http://localhost:8080/api

// Accessing in a React app using dotenv
require('dotenv').config();
const apiUrl = process.env.REACT_APP_API_URL;
```

**Data:**

```

.env.development:

CODE
Copy
REACT_APP_API_URL=http://localhost:8080/api

.env.production:

CODE
Copy
REACT_APP_API_URL=https://api.example.com/v1
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Build Time
6. Use .env files?
7. Customize configurations
8. Use defaults
9. Package application
10. Package application
11. Run Time
12. Access environment variables?
13. Dynamically configure app
14. Use hardcoded values (badpractice)
15. Runtime execution

