---
type: "practice"
difficulty: "Expert"
---

# Design a strategy for an org drowning in microservices (300+, many barely maintained) to consolidate back toward well-sized services. How do you decide what to merge, what to kill, and how do you sell 'fewer services' as progress?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Composition and BFF](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
The challenge here is to streamline a monolithic system that has fragmented into hundreds of poorly maintained microservices. Our goal is to consolidate these microservices into fewer, more efficient and maintainable services without disrupting existing functionalities or degrading performance. This involves assessing service dependencies, identifying duplicated logic, understanding the business processes they support, and determining the risk tolerance for consolidation.

#### Triage
The interviewer is probing our ability to manage complex systems, prioritize based on impact, and effectively communicate value propositions. We need to make strategic decisions that balance technical debt reduction with minimal operational disruptions. Key assumptions include:

The organization has a clear understanding of what each microservice does.
There are no external constraints or dependencies outside the system's control.
The team can assess the current state and define a path forward.

#### First-Principles Solution

Assess Service Health: Evaluate the health, usage metrics, and maintenance status of all services using a dashboard or monitoring tool like Prometheus and Grafana. Identify which services are underutilized, not maintained, or cause frequent issues.
Identify Dependencies: Use tools like Spring Cloud Contract to identify dependencies between microservices. Map out the service topology to understand critical paths.
Determine Duplicates: Analyze codebases for duplicated logic and functionality using static analysis tools such as SonarQube. Merge services with redundant functionalities.
Business Impact Analysis: Collaborate with domain experts (e.g., product managers, business analysts) to map each microservice's contribution to the overall business value. Prioritize consolidation based on this impact.
Risk Assessment: Evaluate the risk of each potential merge or kill scenario using a cost-benefit analysis tool. Consider factors like service downtime, data integrity, and user experience.
Plan Gradually: Implement changes in phases to minimize disruption. For example, start by merging services with low interdependencies first.
Communication Strategy: Develop a clear communication plan to sell the benefits of consolidation to stakeholders. Use metrics such as reduced maintenance costs, improved performance, and enhanced scalability.

#### Closing Summary
By systematically evaluating each microservice, understanding its role within the organization, and aligning our efforts with business goals, we can make informed decisions about merging or killing services. This approach ensures that any changes are well-planned and communicated effectively to all stakeholders.

#### Common Mistakes/Follow-Ups

Failing to adequately assess the impact on existing systems and users.
Overly aggressive consolidation leading to service outages.
Not involving domain experts in decision-making processes, which can result in suboptimal outcomes.
Lack of a clear communication strategy, potentially causing resistance from stakeholders.

**Comparison:**

| Service | Health Score | Usage Metrics       | Maintenance Status | Dependencies     |
|---------|--------------|---------------------|--------------------|------------------|
| A       | 85           | Requests/Min:10, RT:20ms | Well-maintained    | B                |
| B       | 60           | Requests/Min:5, RT:30ms  | Poorly maintained  | -                |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Assess Service Health] --> B[Identify Dependencies]
    B --> C[Determine Duplicates]
    C --> D[Business Impact Analysis]
    D --> E[Risk Assessment]
    E --> F[Plan Gradually]
    F --> G[Communication Strategy]
```

**Example input / output:**

```text
sequenceDiagram
    participant DevOps as Developer
    participant MonitoringTool as Prometheus/Grafana
    participant DependencyTool as Spring Cloud Contract
    participant AnalysisTool as SonarQube
    participant BusinessAnalyst as BA
    participant RiskAssessor as RA

    Developer->>MonitoringTool: Fetch service health metrics
    Note right of MonitoringTool: Service A: High health, low usage
    MonitoringTool->>Developer: HealthScore=85, UsageMetrics={requestsPerMinute:10}

    Developer->>DependencyTool: Identify dependencies
    DependencyTool-->>Developer: Dependencies=[ServiceB]

    Developer->>AnalysisTool: Analyze code for duplicates
    Note right of AnalysisTool: Code duplication found in ServiceA and ServiceC
    AnalysisTool-->>Developer: MergeCandidates=[ServiceA, ServiceC]

    Developer->>BusinessAnalyst: Assess business impact
    BA-->>Developer: CriticalPath={ServiceA:HighImpact}

    Developer->>RiskAssessor: Evaluate risk of changes
    RA-->>Developer: LowRiskChanges=[MergingServiceAandC]
```

**Data:**

```yaml
# Example Data Schema for Service Metrics and Dependencies
services:
  - name: ServiceA
    healthScore: 85
    usageMetrics: { requestsPerMinute: 10, responseTimeMs: 20 }
    maintenanceStatus: "Well-maintained"
    dependencies: [ServiceB]
  - name: ServiceB
    healthScore: 60
    usageMetrics: { requestsPerMinute: 5, responseTimeMs: 30 }
    maintenanceStatus: "Poorly maintained"
    dependencies: []
```

**Flow:**

1. Assess Service Health
2. Identify Dependencies
3. Determine Duplicates
4. Business Impact Analysis
5. Risk Assessment
6. Plan Gradually
7. Communication Strategy

