---
type: "practice"
difficulty: "Expert"
---

# You inherit a shared-database anti-pattern where 30 services reach into each other's tables. As principal, design the multi-year path to data ownership: the data-contract mechanism between producers and consumers, ownership of shared entities, and the review gates that stop it creeping back.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Federated vs Centralized Data Ownership](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Federated_vs_Centralized_Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a significant architectural challenge in distributed systems—specifically, the shared database anti-pattern where 30 services are directly accessing each other's tables. As a Database Developer Architect/Principal, our goal is to refactor and redesign this system into one that enforces data ownership and contract-based interactions between services. The path involves implementing a robust data-contract mechanism, defining clear ownership of shared entities, and establishing review gates to prevent the anti-pattern from re-emerging.

To address these challenges, we'll take a methodical approach:

Understand Current System: Analyze how each service currently accesses other tables.
Identify Shared Entities: Determine which data is being shared and by whom.
Design Data Contracts: Create interfaces or APIs that services will use to interact with the shared entities.
Implement Ownership Rules: Assign clear ownership of shared entities to responsible teams.
Establish Review Gates: Set up a process for reviewing changes to ensure compliance.

Let's dive into each step in detail:

Step 1: Understand Current System

First, we need to understand how each service currently interacts with the database. This involves gathering detailed information on the queries and operations performed by each service. We'll create an inventory of these interactions and identify any dependencies between services that could be problematic.

Step 2: Identify Shared Entities

Next, we'll categorize all entities accessed by multiple services as shared entities. For example, if Service A reads from table X and writes to it, while Service B also accesses table X for reads but not writes, then table X is a shared entity. We'll document these dependencies.

Step 3: Design Data Contracts

Based on the identified shared entities, we'll design data contracts that define how services should interact with each other. These could be REST APIs, gRPC services, or GraphQL schemas depending on the context. The key idea here is to abstract away direct database access and expose only necessary operations via these well-defined interfaces.

For example, if Service A needs to update a record in table X, it would make an API call to a contract managed by Service B that has write access to table X. This ensures that Service A doesn't directly modify the data, thereby reducing coupling and potential issues.

Step 4: Implement Ownership Rules

To enforce data ownership, we'll assign responsibility for each shared entity to specific teams or individuals. For instance, if a team is responsible for managing customer data (shared entities), they would own all operations related to this data. This approach not only helps in accountability but also in maintaining the integrity of the data.

Step 5: Establish Review Gates

Finally, we'll implement review gates that prevent services from regressing to direct database access. This could be done through a combination of automated code reviews and manual approvals before changes can go live. For example, any update to a service that involves direct database queries should undergo a thorough review process.

#### Closing Summary
and Common Mistakes

By following this path, we ensure a gradual transition from shared database tables to well-defined contracts and services with clear ownership rules. The key is maintaining disciplined adherence to these new practices through continuous monitoring and regular code reviews.

#### Common mistakes
include:

Failing to fully document the current state of interactions.
Not clearly defining data contracts and roles early enough.
Overlooking the need for comprehensive review processes.
Underestimating the impact on existing services that are deeply integrated with shared tables.

**Comparison:**

| Service | Dependency Type | Shared Entities | Ownership Team |
|---------|----------------|-----------------|---------------|
| A       | Consumer        | tableX          | Team B        |
| B       | Provider, Consumer | tableX         | Team C        |
| C       | Consumer        | -               | Team D        |

**Code:**

```
MERMAID
Copy
graph LR
    A[Service A] -->|Reads table X| B[Service B]
    B -->|Updates table X| C[Service C]
```

**Example input / output:**

```text
| Service | Action | Entity | Current Implementation |
|---------|--------|--------|-----------------------|
| A       | Read   | tableX | Direct SQL Query      |
| B       | Write  | tableX | Direct SQL Query      |

To refactor:
| Service | Action | Entity | New Implementation    |
|---------|--------|--------|-----------------------|
| A       | Read   | tableX | API Call to B         |
| B       | Write  | tableX | API Call from A and C |
```

**Data:**

```json
{
  "services": {
    "serviceA": {"readEntities": ["tableX"], "writeEntities": []},
    "serviceB": {"readEntities": ["tableX"], "writeEntities": ["tableX"]},
    "serviceC": {"readEntities": [], "writeEntities": []}
  }
}
```

**Flow:**

1. Request for Read
2. Forward Request
3. Service A
4. API Gateway
5. Process
6. Return to
7. Service B
8. Response
9. Send Response
10. API Gateway
11. Service A

