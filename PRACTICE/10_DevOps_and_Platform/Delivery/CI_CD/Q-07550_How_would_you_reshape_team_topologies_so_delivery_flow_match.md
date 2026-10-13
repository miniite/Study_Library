---
type: "practice"
difficulty: "Deep"
---

# How would you reshape team topologies so delivery flow matches the target architecture when Conway's law has team boundaries cutting straight across the end-to-end deploy path, and what stream-aligned versus platform-team split would you propose to make the org ship the system you actually want?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Deployment Gating Trade-offs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Deployment_Gating_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with a situation where Conway's Law is causing issues in our CI/CD pipeline by aligning team boundaries with the end-to-end deployment path. To reshape these topologies and ensure that the delivery flow matches the target architecture, we need to consider stream-aligned teams versus platform teams as two potential organizational structures. The goal is to create an organization that can effectively deliver a system that meets our desired architectural goals.

Firstly, let's restate what the question is probing:

Understanding Conway’s Law: Recognize how team boundaries are influencing the CI/CD pipeline.
Stream-Aligned Teams: Evaluate if these teams can provide end-to-end ownership and thus align better with the target architecture.
Platform Teams: Assess whether platform teams, which focus on shared infrastructure or services, might be a better fit.
Triaging the Problem

Conway's Law states that "organizations which design systems are constrained by the communication structures of those organizations." In our case, team boundaries are cutting across the end-to-end deploy path. This means teams may not have full ownership over their component from development to deployment, leading to potential integration issues and suboptimal delivery flow.

#### First Principles Solution
Stream-Aligned Teams

Stream-aligned teams focus on delivering features or user stories that span multiple parts of the system but are managed by a single team. These teams are responsible for end-to-end ownership, including design, development, testing, and deployment. To make this work, we need to:

Define Streams: Identify streams (sets of related features) based on the target architecture.
Form Teams Around Streams: Create cross-functional teams that own these streams from start to finish.
Reduce Silos: Ensure team members can move between streams as needed, but have clear ownership responsibilities within each stream.
Platform Teams

Platform teams, in contrast, focus on providing shared infrastructure or services that multiple development teams use. This approach is beneficial when different parts of the system need distinct capabilities or when teams are too specialized to own end-to-end processes.

Identify Shared Capabilities: Determine which services or components can be abstracted into platforms.
Create Platform Teams: Form teams dedicated to maintaining and improving these shared services.
Integrate Platforms into Streams: Ensure platform teams provide the necessary tools and infrastructure for stream-aligned teams.

#### Common Mistakes
and Follow-Ups
Over-Specialization: Team members might become too specialized, reducing their ability to understand or participate in other streams.
Communication Overhead: Stream-aligned teams can lead to increased communication overhead as teams need to coordinate across different parts of the system.
Dependency Management: Shared platforms can introduce complexity if not managed properly; dependencies between services must be well-defined and maintained.

#### Closing Summary

Given Conway's Law, reshaping team topologies involves a careful balance. Stream-aligned teams provide end-to-end ownership but require effective coordination and may face communication challenges. Platform teams offer shared infrastructure but need to be designed with clear interfaces and responsibilities. The choice depends on the specific needs of the system, such as complexity, scale, and cross-functional requirements.

#### Common Mistakes
Failing to define clear streams or platforms.
Overlooking the importance of cross-team collaboration when using stream-aligned teams.
Mismanaging dependencies between platform services and stream-aligned development.

**Comparison:**

| Stream Team | Features                  | Dependencies            |
|------------|---------------------------|-------------------------|
| UX         | Login, Home Screen        | Database Service        |
| Payment    | Subscription Management   | Security Service, DB Service |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Stream Team] --> B[Feature A Design];
    A --> C[Feature B Test];
    A --> D[Feature C Deploy];
    E[Platform Team] --> F[Shared Service];
```

**Example input / output:**

| Input (Stream Team) | Output (Platform Service) |
|---------------------|--------------------------|
| Feature A Design     | Database Service          |
| Feature B Test       | Security Service          |
| Feature C Deploy     | -                         |

**Data:**

```json
{
  "streamTeams": [
    {
      "name": "User Experience",
      "features": ["Login", "Home Screen"]
    },
    {
      "name": "Payment Processing",
      "features": ["Subscription Management", "Transaction Handling"]
    }
  ],
  "platformServices": [
    {
      "name": "Database Service",
      "dependencies": []
    },
    {
      "name": "Security Service",
      "dependencies": ["Database Service"]
    }
  ]
}
```

**Flow:**

1. +---------------------------------------------------+
2. | Stream Team                                       |
3. | +----+ +----+ +----+                               |
4. | |UX  | |Pay | |Auth|                               |
5. | |    | |ment| |enti|                               |
6. | |Team| |ment| |ties|                               |
7. | +----+ +----+ +----+                               |
8. |                                                 |
9. |         +-----------+                             |
10. |         | Platform  |                             |
11. |         | Service   |                             |
12. |         +-----------+                             |
13. |         | Database      Security       DB         |
14. |         | Service       Service        Service    |
15. +---------------------------------------------------+

