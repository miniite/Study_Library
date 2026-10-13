---
type: "practice"
difficulty: "Deep"
---

# How do you make a strategic decision to sunset a homegrown platform component that works fine but only one person understands — the classic 'it's not broken but it's a liability' call?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Late and Out-of-order Data Strategy](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Late_and_Out-of-order_Data_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with the decision to sunset a homegrown data platform component that is currently functional but has become a single-point-of-failure due to its complexity and lack of understanding, a strategic approach is essential. This component is often described as "not broken but a liability," meaning it works as intended but poses significant risks in terms of maintenance, scalability, and future development.

To make an informed decision, we need to consider several aspects: the current state of the system, potential risks, alternative solutions, and long-term impacts on the organization. Here's how you would systematically address this challenge:

Assess Current State: Understand the exact functionality, dependencies, and criticality of the component within the broader data ecosystem.
Identify Risks: Evaluate the risks associated with continued use versus potential disruptions from a transition or replacement.
Explore Alternatives: Research existing solutions in the market that could replace or augment the current component.
Evaluate Costs and Benefits: Calculate both direct and indirect costs (e.g., licensing, training) against benefits such as improved maintainability and scalability.

Let’s break down these steps:

Assess Current State

Identify the key functionalities provided by the homegrown component.
Map its dependencies on other systems or components within the data platform.
Determine if there are any compliance or regulatory requirements it fulfills uniquely.

Identify Risks

Single Point of Failure: If this component fails, what is the impact on the entire system? Can you quantify downtime and recovery times?
Maintenance Burden: How much time does maintenance currently take? Is it more efficient to continue with a single maintainer or transition to a team?
Future Scalability: Will this component be able to scale as your data volume grows, or will it become a bottleneck?

Explore Alternatives

Research commercially available solutions that offer similar functionality.
Evaluate open-source alternatives and their community support.

#### Cons
ider cloud-based services if they can provide the necessary features.

Evaluate Costs and Benefits

Calculate the cost of maintenance, including salaries and infrastructure.
Estimate one-time costs for transitioning or replacing the component (e.g., licensing fees, training).
Assess long-term benefits such as reduced risk, improved performance, and easier scalability.

By methodically addressing these aspects, you can make a well-informed strategic decision. In this case, it might be more cost-effective to transition away from the homegrown solution even if it currently works fine. However, the final decision should align with broader organizational goals and stakeholder expectations.

**Comparison:**

| Risk Type            | Description                                                                                           | Impact Level |
|----------------------|-------------------------------------------------------------------------------------------------------|--------------|
| Single Point of Failure | Component failure can cause significant downtime and recovery times.                                 | High         |
| Maintenance Burden   | The current maintainer is the sole point of expertise, leading to potential bottlenecks.               | Medium       |
| Future Scalability    | As data volume grows, the custom component may become a bottleneck, limiting future growth.            | High         |

**Code:**

```text
# Example Python code for assessing component dependencies
def assess_dependencies(component_name):
    # Assume a database connection to retrieve dependency information
    pass

assess_dependencies("custom_data_processor")
```

**Example input / output:**

| Input                | Output                                      |
|----------------------|---------------------------------------------|
| Current component functionality & dependencies assessment | Key risks and alternatives identified       |
| Risks of continued use (e.g., single-point-of-failure)     | Alternative solutions evaluated              |
| Cost-benefit analysis                                       | Strategic decision document generated        |

**Data:**

```json
{
  "dependencies": [
    {"component": "database", "type": "read", "impact_level": "high"},
    {"component": "web_application", "type": "write", "impact_level": "medium"}
  ],
  "risks": [
    {"risk_type": "single_point_of_failure", "description": "Component failure can cause significant downtime."}
  ]
}
```

**Flow:**

1. Assess Current State
2. Evaluate Risks
3. Explore Alternatives
4. Evaluate Costs and Benefits
5. Strategic Decision Document
6. Implement Transition Plan

