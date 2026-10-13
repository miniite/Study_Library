---
type: "practice"
difficulty: "Deep"
---

# You inherit a monolith that must become ~40 services over three years. Lay out the strangler-fig sequencing: which seams you cut first, how you keep the org shipping throughout, and the leading indicators that tell you the decomposition is going wrong.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed ID Generation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_ID_Generation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To transform a monolith into 40+ microservices over three years using strangler-fig sequencing, we must carefully plan which seams to cut first, ensure continuous delivery of working features, and monitor leading indicators to gauge progress and potential issues. The key is to break down the monolith in a way that minimizes disruption while gradually shifting the load towards the new architecture.

First, understand the current system by identifying the core business functions and their dependencies. Map out the interactions between components using tools like dependency graphs or service maps. Next, prioritize high-value seams—those that offer the most immediate benefit from refactoring, such as modules with complex or frequently changing logic.

We will start by cutting seams around these prioritized areas, implementing new services to handle the specific functions. As each microservice is developed, it should be deployed and tested independently in a staging environment before going live. This ensures that any issues can be isolated and resolved without disrupting production.

Continuous integration and delivery (CI/CD) pipelines will be crucial for maintaining stability and velocity. Each deployment must be thoroughly reviewed by the team to ensure it meets quality standards before merging into the main codebase. Automated tests, including unit, integration, and end-to-end tests, should cover every aspect of functionality to catch bugs early.

Leading indicators include metrics such as system availability, response times, and error rates. We need to track these closely throughout the decomposition process. If any of these metrics start to degrade, it may indicate that we are moving too fast or encountering unexpected issues with dependencies between services. Additionally, code coverage and test execution rates provide insights into the stability of our new microservices.

**Comparison:**

```
Leading Indicators	Metric
System Availability	>99% uptime
Response Times	<50ms for critical requests
Error Rates	<1% failure rate in production
Code Coverage	>80% tested code
Test Execution Rate	100% of changes covered in tests before merge
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Monolith] --> B{Cut a seam first?};
    B --> C[New Service A];
    C --> D[Deploy to Staging];
    D --> E[Monitor Metrics];
    E --> F{Issue Detected?};
    F -->|Yes| G[Cause Analysis];
    F -->|No| H[Next Seam];
    G --> I[Collapse or Cut Another Seam?];
    I -->|Collapse| J[Revert Deployment];
    I -->|Cut| K[New Service B];
    K --> L[Deploy to Staging];
```

**Example input / output:**

```text
Input (User logs in):
{
  "username": "user1",
  "password": "pass1"
}

Output (New ServiceA deployed):
{
  "success": true,
  "userRole": "admin"
}
```

**Data:**

```yaml
services:
  - name: ServiceA
    description: Handles user authentication and authorization.
    dependencies:
      - DatabaseService
      - LoggingService
  - name: ServiceB
    description: Manages user sessions and state.
    dependencies:
      - CacheService
      - NotificationService
```

**Flow:**

1. Yes
2. No
3. Collapse
4. Cut
5. Monolith
6. Cut a seam first?
7. New Service A
8. Deploy to Staging
9. Monitor Metrics
10. Issue Detected?
11. Cause Analysis
12. Next Seam
13. Collapse or Cut Another Seam?
14. Revert Deployment
15. New Service B
16. Deploy to Staging

