---
type: "practice"
difficulty: "Deep"
---

# You need to set a company-wide policy on synchronous versus asynchronous inter-service calls so teams stop building fragile blocking chains. How do you enforce it without dictating every integration?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sync vs Async Processing](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Sync_vs_Async_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of setting a company-wide policy on synchronous versus asynchronous inter-service calls, we need to create a robust framework that encourages asynchronous communication while still allowing teams flexibility in their implementation choices. The core idea is to define clear guidelines and best practices that can be enforced through tools like code reviews and automated testing without micromanaging specific integrations.

First, let's break down the question: We are tasked with establishing company-wide policies for inter-service communication that prioritize non-blocking design but allow teams to make informed decisions about their own services. The key is to strike a balance between enforcing standards and empowering development teams.

#### Triage

The interviewer is probing our understanding of cloud fundamentals, specifically around service-oriented architectures (SOA) and the trade-offs between synchronous and asynchronous calls. We need to consider how these choices affect system reliability, scalability, and overall performance. Assumptions include that we are working with modern microservices architecture and have tools like CI/CD pipelines available.

#### First-Principles Solution
Define Clear Best Practices: Establish guidelines for when synchronous or asynchronous calls should be used based on service latency, data complexity, and criticality.
Automated Testing & CI/CD Integration: Implement checks in CI/CD pipelines to ensure adherence to these practices without over-specifying implementation details.
Documentation & Training: Provide detailed documentation and training sessions for teams to understand the rationale behind each guideline and how to apply it effectively.
Step-by-Step Solution

Identify Service Latency Profiles:

Analyze average latencies of services within your ecosystem. Services with higher latencies should generally use asynchronous calls to avoid blocking.

Define Best Practices Document:

Create a comprehensive document outlining when synchronous calls are acceptable (e.g., real-time transactions, critical operations).
Specify that for non-critical or high-latency services, asynchronous communication is preferred.
Include examples and anti-patterns.

Integrate Testing Frameworks:

Use tools like SonarQube to detect synchronous call patterns in code during development stages.
Implement custom rules within your CI/CD pipeline (using Jenkins, GitHub Actions, etc.) that trigger alerts or fail builds when excessive blocking calls are detected.

Educate and Train Teams:

Organize regular training sessions where developers can discuss best practices and share real-world examples.
Encourage peer reviews focusing on adherence to these guidelines.

#### Closing Summary

By defining clear, pragmatic best practices rather than rigid policies, we empower development teams while ensuring the overall architecture remains resilient. The key is to balance flexibility with enforceability through a combination of documentation, automated checks, and continuous education.

#### Common Mistakes/Follow-Ups
Overly Prescriptive Policies: Rigid rules can stifle innovation; always ensure your guidelines are flexible enough for different use cases.
Lack of Automation: Automated testing and CI/CD integrations should be in place to catch issues early without manual oversight.
Insufficient Training: Regular training sessions help ensure all teams understand the importance and application of these policies.

**Comparison:**

```
Service	Latency (ms)	Best Practice
ServiceA	20	Synchronous
ServiceB	150	Asynchronous
ServiceC	300	Asynchronous
```

**Code:**

```python
# Example: Custom Jenkins pipeline stage for checking synchronous calls
def check_synchronous_calls(code):
    # Simple regex-based rule to detect synchronous HTTP calls
    if re.search(r'sync\s+http', code, re.IGNORECASE):
        return "Synchronous call detected in commit. Please review."
    return None

pipeline {
    stages {
        stage('Check for Synchronous Calls') {
            steps {
                script {
                    def result = check_synchronous_calls(currentBuild.changeSets[0].items[0].msg)
                    if (result) {
                        error(result)
                    }
                }
            }
        }
    }
}
```

**Example input / output:**

```text
Input:
- Code commit message containing a synchronous HTTP call

Output:
- Jenkins pipeline stage fails and triggers an alert
```

**Data:**

```json
{
  "synchronousCalls": [
    { "serviceA": true, "serviceB": false },
    { "serviceC": false, "serviceD": true }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Training & Documentation
4. Create Best Practices Doc
5. Train Teams on SOA Principles
6. Regular Code Reviews
7. Peer Review Process
8. Service Latency Analysis
9. Is Service Critical?
10. Allow Synchronous
11. Use Async Communication
12. Document Best Practices
13. Automate Testing & CI/CD

