---
type: "practice"
difficulty: "Intermediate"
---

# How do you keep an AI assistant from confidently writing code against the wrong environment config (dev vs prod) when the distinction lives in your head?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Trust Boundaries for AI Generated Code](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Trust_Boundaries_for_AI_Generated_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The key challenge is to ensure that the AI assistant correctly identifies and applies configuration settings based on whether it's operating in a development or production environment, without relying solely on manual oversight. This involves implementing robust naming conventions, leveraging environment-specific variables, and integrating dynamic checks within the code.

#### Triaging the Question

This question explores how to prevent an AI from generating incorrect configurations by writing prod code in dev environments (or vice versa). The primary concerns are ensuring consistency, avoiding security risks, and maintaining code integrity. We need to consider both technical and procedural safeguards.

#### Assumptions
Environment Configuration: Dev and Prod use different configuration files or variables.
Dynamic Checks: Code can be designed to perform runtime checks to ensure it’s running in the correct environment.
Naming Conventions: Consistent naming for resources and configurations helps prevent accidental misuse.

#### First-Principles Solution
Step 1: Environment-Specific Variables

Use distinct environment-specific variables for configuration values. For example:

PLAINTEXT
// .env file
DEV_API_URL=https://dev.example.com/api
PROD_API_URL=https://prod.example.com/api

# Code snippet

#### cons
t apiUrl = process.env.NODE_ENV === 'production' ? process.env.PROD_API_URL : process.env.DEV_API_URL;
Step 2: Runtime Environment Check

Implement a runtime check to enforce the correct environment. This can be done using:

JAVASCRIPT
if (!process.env.NODE_ENV || (process.env.NODE_ENV !== 'development' && process.env.NODE_ENV !== 'production')) {
    throw new Error("Environment variable NODE_ENV must be set to either 'development' or 'production'");
}
Step 3: Code Reviews and Automated Tests

Ensure that all code changes are reviewed by peers, and automated tests (unit and integration) cover both dev and prod scenarios.

Step 4: Naming Conventions

Use clear and consistent naming for environment-specific resources. For example:

PLAINTEXT
// Dev vs Prod Database Names
dev-db.example.com
prod-db.example.com

// Code snippet

#### cons
t dbName = process.env.NODE_ENV === 'production' ? 'prod-db' : 'dev-db';
Step 5: CI/CD Pipeline Integration

Integrate the environment check into your Continuous Integration and Continuous Deployment (CI/CD) pipeline. This ensures that code changes are validated before deployment.

#### Closing Summary

By implementing these measures, we can mitigate the risk of the AI assistant generating incorrect configurations by ensuring it operates within the correct environment context. Consistent practices and automated checks help maintain code integrity and security.

#### Common Mistakes
and Follow-Ups
Manual Overlook: Failing to set environment variables correctly or missing a manual step in deployment.
Overreliance on CI/CD: Assuming that all issues will be caught by static analysis tools without dynamic runtime checks.
Lack of Code Reviews: Not having a peer review process for code changes can lead to overlooked errors.

#### Example
of a Common Mistake

If the AI assistant generates code based on prod configurations but it’s running in dev, this could result in incorrect behavior or security vulnerabilities (e.g., using sensitive prod API endpoints in development).

Triaging Further

You might also explore:

AI Interventions: Using AI to suggest changes that are environment-aware.
User Education: Training users on the importance of environment separation and best practices.

