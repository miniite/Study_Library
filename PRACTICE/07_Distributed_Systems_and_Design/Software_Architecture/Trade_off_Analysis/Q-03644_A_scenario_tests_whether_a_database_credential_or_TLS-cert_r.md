---
type: "practice"
difficulty: "Deep"
---

# A scenario tests whether a database credential or TLS-cert rotation across the entire fleet can complete before expiry with no connection interruption. Walk through the automation, staggering, and rollback sensitivity points, and where rotation becomes a security-vs-availability tradeoff.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quality Attribute Trade-off Scenarios](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Quality_Attribute_Trade-off_Scenarios.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering the scenario of rotating database credentials or TLS certificates across an entire fleet, we must carefully balance security with availability to ensure minimal downtime during the transition. This process involves several key steps and trade-offs that need to be analyzed through Architectural Trade-off Analysis (ATAM).

Firstly, ATAM helps us understand the various components involved in the rotation process: the database servers, application servers, and any relevant middleware or load balancers. Each of these components must handle credential changes without breaking connections. The goal is to perform these rotations incrementally to minimize risk, while ensuring that no single failure point can cause a full system outage.

We start by identifying the critical paths in our architecture: which services depend on each other and how they communicate. For instance, if we use an application server to connect to multiple databases, rotating credentials for one database might require adjusting configurations in all application instances. This interconnectedness means that any failure in one part of the system can cascade and affect others.

To achieve a smooth rotation, we must plan carefully:

Automation: Develop scripts or tools that handle the credential or certificate changes seamlessly.
Staggering: Implement a staggered approach where only a subset of nodes are updated at a time to monitor for issues before rolling out to all nodes.
Rollback Mechanisms: Ensure there’s a mechanism to revert changes quickly if something goes wrong.

Now, let's break down the steps and potential trade-offs:

Step-by-Step Solution

Identify Dependencies:

Analyze the dependencies between services (e.g., application -> database).
Determine which components need to be updated first and which can wait until later in the rotation cycle.

Automation Scripting:

Write scripts that handle credential or certificate changes, ensuring they are idempotent.
Integrate these scripts with existing CI/CD pipelines for consistent application across environments.

Staggered Rotation:

Split nodes into groups and update one group at a time.
Monitor each group post-rotation to ensure no issues arise before proceeding to the next group.

#### Example
If you have 100 servers, rotate 10% per day for 10 days.

Health Checks:

Implement health checks after each rotation to detect any anomalies quickly.
Ensure monitoring tools are in place to alert on potential issues early.

Rollback Plan:

Have a clear rollback plan and process documented.
Test the rollback mechanism thoroughly before starting the rotation process.
Security vs Availability Tradeoffs
Security: Rotating credentials ensures that even if a compromised key is used, it will only be valid for a short period.
Availability: A staggered approach minimizes the risk of availability issues but increases complexity. If not managed correctly, partial failures can lead to cascading outages.

#### Common Mistakes
and Follow-Ups
Over-Rotation: Rotating credentials too frequently can introduce unnecessary risk if done without adequate planning.
Insufficient Staggering: Failing to stagger rotations properly might result in a sudden outage when multiple services are updated simultaneously.
Inadequate Monitoring: Not having robust monitoring in place can lead to late detection of issues, causing delays and potential outages.

#### Example Scenario

Suppose we have 100 application servers and need to rotate database credentials. We decide to use a staggered approach:

CODE
Day 1: Rotate 10% (10) nodes
Day 2-3: Monitor these nodes for issues
Day 4-5: Rotate another 10% (10 more)
...
Day 98-100: Final 10%

During each rotation, we monitor the application servers closely. If any issue is detected, we can quickly revert to the previous state.

#### Conclusion

The success of rotating database credentials or TLS certificates hinges on a well-planned and executed strategy that balances security with availability. By automating the process, staggering updates, and having a solid rollback plan, we can minimize risks while ensuring minimal disruption during the transition period.

**Comparison:**

```
Step 1: Identify Dependencies
Step 2: Automation Scripting
Step 3: Staggered Rotation
Step 4: Health Checks
Step 5: Rollback Plan
```

**Code:**

```python
# Example Python script for rotating database credentials

def rotate_credentials(nodes):
    for node in nodes:
        # Update credentials on the node
        update_node_credentials(node)
        # Perform health check after updating
        if not perform_health_check(node):
            rollback(node)

def update_node_credentials(node):
    # Logic to update credentials on the node
    pass

def perform_health_check(node):
    # Logic to check node's health after credential update
    return True  # Placeholder for actual logic

def rollback(node):
    # Logic to revert changes if issues arise
    pass

# Staggered rotation example
for day in range(1, 101):
    nodes_to_rotate = calculate_nodes_to_rotate(day)
    rotate_credentials(nodes_to_rotate)

def calculate_nodes_to_rotate(day):
    total_nodes = 100
    return int(total_nodes * (day / 100))
```

**Example input / output:**

| Day | Nodes to Rotate | Health Check Results |
| --- | -------------- | -------------------- |
| 1   | [1, 2]         | Healthy              |
| 2   | [3, 4]         | Healthy              |
| ... | ...            | ...                  |
| 98  | [97, 98]       | Healthy              |
| 99  | [99, 100]      | Healthy              |

**Data:**

```json
{
  "nodes": [
    {"id": 1, "status": "healthy"},
    {"id": 2, "status": "healthy"},
    ...
    {"id": 100, "status": "healthy"}
  ]
}
```

**Flow:**

1. Health Checks
2. Yes
3. No
4. Start
5. Identify Dependencies
6. Automation Scripting
7. Health Check?
8. Rollback if Necessary
9. Proceed to Next Nodes
10. End
11. Check Node Status
12. Log Error if Issue Detected

