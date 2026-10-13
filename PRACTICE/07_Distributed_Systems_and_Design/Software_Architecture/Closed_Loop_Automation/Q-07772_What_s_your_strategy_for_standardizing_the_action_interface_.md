---
type: "practice"
difficulty: "Expert"
---

# What's your strategy for standardizing the action interface (a common 'verb' abstraction) so remediations are portable across clouds and tools?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Desired State Reconciliation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Desired_State_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that remediations can be portable across different cloud environments and tools, we need to define a standardized action interface, commonly referred to as an "action verb" abstraction. This strategy involves several key steps: identifying common operations, creating a standard API layer, integrating this with existing systems, and continuously refining the abstraction based on feedback.

First, let's break down what the interviewer is probing:

Common Operations: We need to identify the most frequent remediation actions performed across different clouds and tools.
Standard API Layer: This involves defining a common interface that abstracts away vendor-specific details.
Integration: Ensuring this standardization can be applied seamlessly across existing systems.
Continuous Refinement: The process must be iterative, with regular updates to the abstraction as new operations emerge or become more critical.

#### Assumptions

The remediation actions are known and consistent enough for a common interface to be viable.
We have access to a diverse set of tools and cloud environments to test and validate the standardization.
There is a willingness from both tool providers and end-users to adopt this standardization.

Step-by-step solution:

Identify Common Operations:

Analyze logs, incidents, and known remediation actions across different clouds and tools to identify recurring patterns.
For example, common operations might include restarting services, changing configuration settings, scaling resources, or applying patches.

Define the Action Verb Abstraction:

Create a set of well-defined action verbs that represent these common operations (e.g., RESTART, CONFIGURE, SCALE, APPLY_PATCH).
For each verb, define a minimal set of parameters required for it to be actionable across different environments. This could include resource identifiers, settings, and other contextual information.

Create a Standard API Layer:

Develop a middleware layer that implements these action verbs as standardized functions.
Ensure this layer can receive input in a structured format (e.g., JSON) and generate output in the same or compatible formats.
Use common libraries or frameworks to facilitate interoperability.

Integrate with Existing Systems:

Modify existing systems to use this standard API layer for performing remediations.
Implement adapters that can translate between vendor-specific APIs and our standardized verbs if needed.
Conduct thorough testing to ensure compatibility and reliability across all integrated systems.

Continuous Refinement:

Regularly collect feedback from users and tool providers on the effectiveness of the standard interface.
Update the action verb set as new operations emerge or existing ones change.
Enhance performance, security, and usability based on real-world usage data.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

By defining a common "verb" abstraction for remediations, we can achieve significant portability across different clouds and tools. However, some common pitfalls include not adequately covering edge cases, over-engineering the initial standardization, or neglecting to involve key stakeholders in the process. Continuous refinement is crucial as the landscape of cloud services evolves.

For follow-ups, you might be asked:

How do you ensure backward compatibility with existing systems?
What are potential security implications and how can they be mitigated?
How would you handle vendor-specific features that don't fit neatly into a common abstraction?

**Comparison:**

```
Verb	Parameters
RESTART	service_id
CONFIGURE	resource_id, settings
SCALE	resource_id, scale_factor
APPLY_PATCH	patch_id
```

**Code:**

```python
# Example of a simplified action verb definition in Python

def restart_service(service_id):
    """
    Restart a service with the given ID.

    Args:
        service_id (str): Identifier for the service to be restarted.
    """
    # Implementation details vary based on cloud provider and environment
    pass

def configure_resource(resource_id, settings):
    """
    Configure a resource with specific settings.

    Args:
        resource_id (str): Identifier for the resource to be configured.
        settings (dict): Dictionary of configuration settings.
    """
    # Implementation details vary based on resource type and cloud provider
    pass

# Example usage
restart_service("service-01")
configure_resource("resource-02", {"max_connections": 50})
```

**Example input / output:**

```text
Input:
{
  "verb": "RESTART",
  "service_id": "service-01"
}

Output:
{
  "status": "success",
  "message": "Service restarted successfully."
}
```

**Data:**

```json
{
  "action_verbs": {
    "RESTART": ["service_id"],
    "CONFIGURE": ["resource_id", "settings"],
    "SCALE": ["resource_id", "scale_factor"],
    "APPLY_PATCH": ["patch_id"]
  }
}
```

**Flow:**

1. Yes
2. Identify Common Operations
3. Is the operation common?
4. Define Action Verb
5. Create Standard API Layer
6. Integrate with Existing Systems
7. Continuous Refinement

