---
type: "practice"
difficulty: "Intermediate"
---

# How would you convince a skeptical partner-engineering leader to accept a mandatory API certification program before partners go live? Design the certification gates, and address their fear that it kills partner-acquisition velocity.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Lifecycle Metrics and Retirement](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Lifecycle_Metrics_and_Retirement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When facing a skeptical engineering leader from a partner, our goal is not just to implement an API certification but to build a robust case that ensures long-term quality and security while minimizing disruption to acquisition processes. Here’s how we can approach this:

TL;DR: We will design a tiered API certification program with clear gates to ensure partners meet the necessary standards for high-quality, secure, and reliable API usage. The key is to show how this aligns with their goals without slowing down partner onboarding.

Restate & Triage:

The core question here is understanding why our proposed certification program (which might seem like a bottleneck) could be beneficial in the long run.
We need to address concerns around potential delays and ensure that partners are onboarded efficiently while maintaining high standards.

#### Assumptions
include that we have visibility into partner API usage, can measure their compliance, and will provide adequate support during the certification process.

#### First-Principles Solution

Define Clear Gating Mechanisms: We need to establish clear criteria for certification. These might include code reviews, security audits, documentation standards, performance benchmarks, and regular maintenance.
Tiered Approach: Implement a tiered certification system where partners can move up the ladder based on their capabilities and commitments.
Comprehensive Training & Support: Provide extensive training sessions and continuous support to help partners meet these standards without significant delays.

Detailed Steps:

Gating Criteria:
Code Quality & Security: Partners must pass code reviews by our security team, ensuring no known vulnerabilities exist.
Documentation Standards: Ensure all APIs have comprehensive documentation, including usage examples and best practices.
Performance Benchmarks: Partner APIs must meet performance benchmarks set by us to ensure they can handle the expected load without issues.
Tiered Certification:
Initial Onboarding: Partners start with basic requirements like API documentation and initial code review.
Advanced Levels: As they demonstrate competence, they advance through additional security audits and performance tests.
Support & Training:
Offer regular training sessions on best practices in API development and security.
Provide one-on-one support during the certification process.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

By implementing a well-structured certification program, we can ensure that partners deliver high-quality APIs which are secure and reliable. This will not only protect our own systems but also enhance partner confidence in using our platform.
The key mistake is to focus solely on the immediate impact without considering long-term benefits. We need to demonstrate how this process streamlines the overall development lifecycle, making it easier for partners once they understand the value.

Common follow-ups might include:

How we measure compliance and ensure continuous improvement?
What’s our plan for handling exceptional cases where partners are unable to meet certain criteria?

**Comparison:**

```
Certification Tier	Gating Criteria	Status
1	Code Quality & Security, Documentation Standards	In Progress
2	Performance Benchmarks	Not Started
3	Comprehensive Maintenance	Not Started
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Initial Onboarding] --> B[Code Quality & Security];
    B --> C[Documentation Standards];
    C --> D[Performance Benchmarks];
    D --> E[Tier 1 Certification];
    E --> F[Tier 2 Certification];
    F --> G[Tier 3 Certification];
    F --> H[Continuous Support];
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Partner requests onboarding | Partner undergoes initial onboarding and moves through certification tiers based on their performance. |

**Data:**

```json
{
  "gates": [
    {
      "name": "Code Quality & Security",
      "description": "Pass code reviews by our security team.",
      "status": false
    },
    {
      "name": "Documentation Standards",
      "description": "Ensure comprehensive documentation with usage examples and best practices.",
      "status": false
    },
    {
      "name": "Performance Benchmarks",
      "description": "Meet performance benchmarks set by us.",
      "status": false
    }
  ]
}
```

**Flow:**

1. Partner Onboarding
2. Initial Code Review
3. Security Audit
4. Documentation Check
5. Performance Test
6. Tier 1 Certification
7. Tier 2 Certification
8. Continuous Support

