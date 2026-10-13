---
type: "practice"
difficulty: "Expert"
---

# How do you run an ATAM when the architecture spans systems owned by three different vendors and you cannot see inside their internals? What scenarios still make sense and where does the method break down?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ ATAM for Build vs Buy](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/ATAM_for_Build_vs_Buy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To conduct an Architectural Trade-off Analysis Method (ATAM) when dealing with a distributed architecture managed by three different vendors, we must focus on leveraging observable behaviors and interfaces while setting clear communication channels to align expectations and gather necessary information. This approach requires us to break down the problem systematically and consider both technical and non-technical aspects.

Firstly, ATAM is designed to evaluate architectural decisions based on multiple perspectives: functional, performance, security, ease of change, maintainability, and cost. In this scenario, we can still apply these dimensions by focusing on external interactions and observable behaviors from each vendor's system.

Define the Scope and Objectives: Clearly define what aspects of the architecture need to be analyzed. For instance, if security is a critical concern, focus on how data flows between systems and the security protocols in place.

Identify External Interfaces: Map out the interfaces that each vendor's system exposes. This includes APIs, databases, message queues, and other forms of communication. Analyze these interfaces to understand their behavior and constraints.

Gather Requirements and Assumptions: Work closely with stakeholders from all vendors to gather requirements and assumptions about how systems should interact. Use this information to create a scenario-based approach where each vendor's system is treated as an opaque black box.

Conduct Walkthroughs and Scenario-Based Testing: Perform walkthroughs of common use cases across the architecture. Create test scenarios that cover typical user journeys, failure modes, and edge cases. Use these scenarios to identify potential trade-offs in terms of functionality, performance, security, etc.

Analyze Interdependencies and Constraints: Identify dependencies between systems and constraints imposed by vendor-specific implementations. For example, a vendor might have strict limitations on the frequency or volume of API calls it can handle. These need to be factored into your analysis.

Evaluate Cost Implications: Consider both direct and indirect costs associated with each architectural decision. This includes licensing fees, operational overhead, and potential future maintenance costs.

#### Cons
ider Scalability and Performance: Assess how well the architecture scales under different loads and performs in various scenarios. Since you cannot see inside the vendors' systems, rely on empirical data gathered from real-world usage or load testing.

Mitigate Vendor Lock-In Risks: Evaluate strategies to minimize vendor lock-in risks, such as implementing standards-based interfaces, using middleware for abstraction, or developing fallback plans.

Document Findings and Recommendations: Summarize the key findings of your ATAM, highlighting areas where trade-offs are necessary. Provide actionable recommendations that can be discussed with stakeholders from all vendors to ensure alignment on future development priorities.

This method breaks down when there is no clear communication channel among the vendors or if one vendor’s system has hidden complexities that significantly impact other parts of the architecture. In such cases, the lack of visibility might lead to incomplete trade-off analysis, potentially resulting in suboptimal architectural decisions.

**Comparison:**

```
Scenario	Description
User Authentication Flow	Steps involved when a user logs in, including API calls to VendorA and VendorB.
Data Synchronization	Process of syncing data between VendorA and VendorB systems.
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Define Scope and Objectives] --> B{Identify External Interfaces};
    B --> C[Map Out Interfaces];
    C --> D[Gather Requirements and Assumptions];
    D --> E{Conduct Walkthroughs and Scenario-Based Testing};
    E --> F[Evaluate Interdependencies and Constraints];
    F --> G[Evaluate Cost Implications];
    G --> H[Consider Scalability and Performance];
    H --> I[Mitigate Vendor Lock-In Risks];
    I --> J[Document Findings and Recommendations];
```

**Example input / output:**

```text
Request:
GET /users

Response:
200 OK
{
  "id": 1,
  "username": "example_user",
  "email": "user@example.com"
}

---

Request:
POST /messages
Content-Type: application/json

{
  "message": "Hello from VendorB"
}

Response:
201 Created
{
  "messageId": 3456789,
  "timestamp": "2023-10-01T14:30:00Z"
}
```

**Data:**

```json
{
  "interfaces": [
    {
      "name": "VendorAAPI",
      "description": "Exposes CRUD operations for user management"
    },
    {
      "name": "VendorBQueue",
      "description": "Sends and receives messages in a real-time streaming platform"
    }
  ],
  "scenarios": [
    {
      "name": "User Authentication Flow",
      "description": "Steps involved when a user logs in, including API calls to VendorA and VendorB."
    },
    {
      "name": "Data Synchronization",
      "description": "Process of syncing data between VendorA and VendorB systems."
    }
  ]
}
```

**Flow:**

1. Define Scope
2. Identify External Interfaces
3. Map Out Interfaces
4. Gather Requirements andAssumptions
5. Conduct Walkthroughs andScenario-Based Testing
6. Evaluate Interdependenciesand Constraints
7. Evaluate Cost Implications
8. Consider Scalability andPerformance
9. Mitigate Vendor Lock-In Risks
10. Document Findings andRecommendations

