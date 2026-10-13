---
type: "practice"
difficulty: "Expert"
---

# Your company acquired three startups each with its own stack, auth, and data model — architect a two-year integration plan that unifies identity and billing without freezing product delivery.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Service Mesh Adoption](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Service_Mesh_Adoption.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To integrate three startups under one unified system while ensuring continued product delivery, we need to design an architecture that aligns with the principles of modularization, microservices, and gradual evolution. The goal is to unify identity management (auth) and billing systems without disrupting ongoing development and deployment cycles.

#### Triaging the Question

The interviewer is probing your understanding of:

Modular Architecture: How to break down large systems into smaller, manageable modules.
Microservices: Techniques for designing loosely coupled services that can be independently developed, deployed, and scaled.
Identity Management: Strategies to unify authentication across multiple stacks.
Billing System Integration: Approaches to consolidate billing logic while preserving data integrity.

#### Assumptions
Each startup has a unique stack, database schema, and user base.
The startups have varying levels of technical debt and code quality.
There are ongoing product development cycles that must be minimally impacted during the integration process.

#### Solution
from First Principles
Identify Commonalities: Start by identifying commonalities across the three startups, such as shared data models or services (e.g., logging, monitoring).
Define Boundaries: Clearly define service boundaries to ensure loose coupling and modularity.
Unified Identity Management: Implement a single identity provider (IdP) that can handle authentication for all three systems.
Billing System Integration: Develop a unified billing system that can aggregate data from different sources.
Migration Strategy: Gradually migrate existing services and databases to the new architecture.
Step-by-Step Solution
1. Identify Commonalities
Shared Services: Determine if there are shared services like logging, monitoring, or user management that can be common across all startups.
Data Model Analysis: Analyze the existing data models to identify any common entities or relationships.
2. Define Boundaries
Service Decomposition: Break down each startup's system into microservices based on their functionality (e.g., frontend, backend services).
API Gateway: Implement an API gateway that routes requests to the appropriate microservices.
Communication Protocol: Use a consistent communication protocol (e.g., REST, gRPC) across all services.
3. Unified Identity Management
IdP Selection: Choose or develop a Single Sign-On (SSO) solution that can manage identities for all users.
OAuth/OIDC: Implement OAuth/OpenID Connect to enable seamless authentication and authorization across different systems.
Token Management: Use JWT tokens for secure user authentication.
4. Billing System Integration
Data Aggregation: Design a microservice responsible for aggregating billing data from each startup's system.

#### Cons
istent Metrics: Ensure consistent metrics are captured and stored in a unified database (e.g., PostgreSQL, Cassandra).
Billing Logic: Develop flexible billing logic that can handle different pricing models and payment methods.
5. Migration Strategy
Phased Approach: Begin by migrating one startup at a time to minimize disruptions.
Parallel Deployment: Deploy the new architecture in parallel with the existing systems until full migration is complete.
Rollback Plan: Develop a robust rollback plan in case of any issues during the transition.

#### Closing Summary
and Common Mistakes

To successfully integrate these startups, it’s crucial to maintain a modular approach that allows for gradual evolution. Common mistakes include:

Over-engineering: Rushing into full redesign without considering incremental improvements.
Underestimating Technical Debt: Failing to address existing technical debt can lead to slower integration cycles.
Lack of Planning: Without proper planning and communication, the migration might face unexpected challenges.

#### Follow-Ups
What are some potential roadblocks during this transition?
How would you ensure data consistency across the unified system?
Can you discuss a similar project you have worked on before?

**Comparison:**

```
Service	Functionality	Technology Stack
Auth Microservice	User authentication, authorization	Node.js, OAuth/OIDC, JWT
Billing Aggregator	Data aggregation and storage	Python, PostgreSQL, gRPC
Monitoring Service	Logging and performance monitoring	Prometheus, Grafana
```

**Code:**

```
MERMAID
Copy
graph TD
    A[API Gateway] --> B(Backend Service 1)
    A --> C(Backend Service 2)
    A --> D(Backend Service 3)
    B --> E[Database]
    C --> E
    D --> E
```

**Example input / output:**

| Input | Output |
|-------|--------|
| User logs into system A with credentials | SSO provider authenticates and returns JWT token |
| Service makes API call to billing microservice | Response containing aggregated billing data for user |

**Data:**

```yaml
# Sample Database Schema

users:
  id: integer
  email: string
  roles: [role1, role2]

billing_transactions:
  user_id: integer
  amount: float
  timestamp: datetime
  service: string
```

**Flow:**

1. User Login
2. SSO Provider Authentication
3. JWT Token Issued
4. API Gateway
5. Backend Service
6. Billing Data Aggregation
7. Billing Response Sent

