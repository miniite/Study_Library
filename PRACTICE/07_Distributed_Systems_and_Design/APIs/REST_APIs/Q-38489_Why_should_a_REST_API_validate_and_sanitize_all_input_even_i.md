---
type: "practice"
difficulty: "Basic"
---

# Why should a REST API validate and sanitize all input even if the frontend already validates it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of URLs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Keeping_Secrets_out_of_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a REST API, validating and sanitizing inputs is an essential practice, even if the frontend has its own validation logic. This approach ensures that the backend remains robust, secure, and maintains data integrity regardless of potential issues in the frontend. In this answer, we will explore why such validation is crucial from first principles.

#### Triage
Understanding the Question

The question is probing your understanding of API security and reliability. It highlights a common practice where both the frontend and backend validate inputs but asks whether it's necessary for the backend to perform these checks as well. Key assumptions include:

The frontend performs initial validation to improve user experience.
Backend validation ensures data integrity, security, and consistency.

#### First-Principles Solution
Security: Frontend validations can be bypassed or manipulated by users through various means like advanced browser tools, direct API calls, or malicious scripts. Backend validation acts as a second line of defense to protect the system from such attacks.
Data Integrity: Validation on both sides ensures that all data passed between client and server is correct and safe for processing. This prevents errors and maintains consistency in database operations.
Isolation and Dependency Management: Relying solely on frontend validation can lead to dependency issues if the frontend codebase changes or if new vulnerabilities are discovered. Backend validation provides a more reliable and consistent check.

Let's break it down step-by-step:

Initial Validation by Frontend:
The frontend performs client-side validation to provide immediate feedback to users and improve performance by reducing unnecessary API calls.
Backend Validation:
Validate data again at the backend to ensure that even if an attacker bypasses or manipulates the frontend, the request is still handled safely.

#### Closing Summary

In summary, validating and sanitizing inputs in a REST API is crucial because it ensures data integrity, enhances security, and provides a consistent mechanism for handling requests. Even though the frontend validates inputs, backend validation serves as an important safeguard against potential attacks and errors.

#### Common Mistakes
and Follow-Ups:

Not performing any backend validation can lead to vulnerabilities such as injection attacks or data corruption.
Over-relying on frontend validation without corresponding backend checks can result in unnecessary risk exposure.
Neglecting edge cases and complex input scenarios during backend validation can leave the system vulnerable.

