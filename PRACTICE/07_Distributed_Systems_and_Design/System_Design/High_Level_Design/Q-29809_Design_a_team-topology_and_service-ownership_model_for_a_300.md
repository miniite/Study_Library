---
type: "practice"
difficulty: "Expert"
---

# Design a team-topology and service-ownership model for a 300-engineer org so that no service is orphaned, on-call is sustainable, and platform teams don't become a bottleneck — sketch the topology.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Service Mesh Adoption](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Service_Mesh_Adoption.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing a high-level design (HLD) for a 300-engineer organization, we need to ensure that no service becomes orphaned, on-call shifts are sustainable, and platform teams don’t become a bottleneck. The key is to balance ownership, communication, and scalability. We'll start by breaking down the problem into components: service ownership, team topology, and on-call strategy.

#### Triage

The core of this question revolves around designing an effective organizational structure that supports high reliability and maintainability in a large-scale engineering organization. Specifically:

Service Ownership: Ensuring every service has clear responsibility and accountability.
Team Topology: Organizing teams to efficiently manage services without creating bottlenecks.
On-call Strategy: Maintaining sustainable on-call rotations.

#### Assumptions
The organization is divided into multiple services, each critical for the overall functionality of the system.
There are various types of engineers (e.g., frontend, backend, database) with different skill sets.
The organization aims to maintain a high level of reliability and availability across all services.
Step-by-Step Solution

Service Ownership:

Each service should be owned by a specific team or group of teams.
This ownership ensures accountability for the health, performance, and evolution of the service.

Team Topology:

Organize teams around services to form domain-specific groups (e.g., a "Customer Service" team owns all services related to customer interactions).
Include cross-functional skills within each team (developers, testers, operators) to ensure holistic ownership.

On-call Strategy:

Implement an on-call rotation that is distributed and sustainable.
Ensure no single service or domain is overburdened with on-call responsibilities.
Detailed Topology
1. Service Ownership
Each service should be owned by a cross-functional team consisting of developers, testers, and operators.

#### Example
A "Customer Service" domain would have multiple microservices (e.g., authentication, user management, billing). Each microservice is owned by the corresponding sub-team.
2. Team Topology
Domain Teams: Organize teams based on business domains rather than traditional tech silos.
For instance: CustomerServiceTeam, PaymentServiceTeam, etc.
Cross-cutting Concerns: Establish platform teams that provide common infrastructure support (e.g., logging, monitoring, security).

#### Example
A "Platform Team" that provides shared services like logging and monitoring.
3. On-call Strategy
Implement a distributed on-call schedule where each domain team rotates on-call responsibilities.
Ensure rotation periods are manageable to prevent burnout.
Use tools to manage and track on-call shifts (e.g., PagerDuty, Opsgenie).

#### Closing Summary
and Common Mistakes/Follow-ups

In summary, the key is to balance ownership with cross-functional skills and ensure sustainable on-call rotations. Domain teams provide focused expertise while platform teams offer shared infrastructure support.

#### Common Mistakes

Over-reliance on technical silos (e.g., all backend engineers owning database services).
Inefficient on-call rotations leading to burnout.
Understaffed or overburdened domain teams.

#### Follow-ups

How would you measure the success of this model?
What steps would you take to ensure platform teams remain non-bottlenecking?
How do you handle cross-domain issues that span multiple services?

**Comparison:**

| Team Name          | Description                             | Ownership Model  |
|--------------------|-----------------------------------------|------------------|
| CustomerServiceTeam| Owns services related to customer       | Cross-functional |
| PaymentServiceTeam  | Owns services related to payments       | Cross-functional |
| PlatformTeam       | Provides shared infrastructure support | Cross-functional |

**Code:**

```
MERMAID
Copy
graph TD;
    A[CustomerServiceTeam] --> B[AuthenticationMicroservice];
    A --> C[BillingMicroservice];
    A --> D[UserManagementMicroservice];

    P[PlatformTeam] --> E[Logging];
    P --> F[Monitoring];
```

**Example input / output:**

| Input | Output |
|-------|--------|
| On-call schedule for `CustomerServiceTeam` | Rotated on-call shifts with clear responsibilities. |
| Monitoring logs from `AuthenticationMicroservice` | Aggregated and visible via the PlatformTeam's monitoring system. |

**Data:**

```yaml
services:
  - name: AuthenticationMicroservice
    owner_team: CustomerServiceTeam
  - name: BillingMicroservice
    owner_team: CustomerServiceTeam
  - name: UserManagementMicroservice
    owner_team: CustomerServiceTeam

platform_services:
  - name: Logging
    provider: PlatformTeam
  - name: Monitoring
    provider: PlatformTeam
```

**Flow:**

1. On-call Shift
2. CustomerServiceTeam
3. Handle Customer Issue
4. Log & Monitor
5. PlatformTeam
6. Support and Scale Monitoring

