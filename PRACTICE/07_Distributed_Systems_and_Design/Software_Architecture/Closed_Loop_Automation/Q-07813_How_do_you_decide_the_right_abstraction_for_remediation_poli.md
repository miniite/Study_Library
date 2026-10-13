---
type: "practice"
difficulty: "Deep"
---

# How do you decide the right abstraction for remediation policies — DSL, declarative config, or code — for an org with mixed engineering maturity?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Desired State Reconciliation](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Closed_Loop_Automation/Desired_State_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In a large organization (Org) with diverse engineering capabilities, deciding the right abstraction level for remediation policies is critical to balancing ease of use and operational efficiency. The key choices are between Domain-Specific Languages (DSLs), declarative configurations, or code-based solutions. This decision hinges on factors such as the maturity of the engineers, the complexity of the system, and the desired speed of deployment.

#### Triage

The core question here is about selecting the most appropriate abstraction layer for remediation policies based on the engineering maturity of the organization. The interviewer is probing your understanding of how different levels of abstraction impact maintainability, flexibility, and the overall operational efficiency of the AIOps system. You need to consider assumptions such as the current state of DevOps practices, the technical skills of the team, and the organizational culture around automation.

#### First-Principles Solution

Understanding Engineering Maturity:

Assess the current skill levels and preferences of the engineering teams.
Identify whether the majority is proficient in scripting (e.g., Python) or if they are more comfortable with declarative tools.

Evaluating Complexity:

Determine the complexity of the remediation logic required. Simple cases can often be handled by DSLs, while complex scenarios might necessitate code-based solutions.

#### Cons
ider whether the policies need to be frequently updated or remain static over time.

Flexibility and Maintainability:

DSLs offer a balance between flexibility and ease of use for non-developers.
Declarative configurations provide clear and unambiguous instructions, ideal for complex scenarios that require consistency.
Code-based solutions give the highest level of control but may be overkill for simpler cases.

Deployment Speed:

Faster deployment cycles might favor DSLs or declarative configurations due to their ease of use.
For more critical systems, code-based solutions can ensure high-quality and robust remediation logic with fewer issues during deployment.
Choosing the Right Abstraction

Based on these factors:

For Maturity Level 1 (Beginner/Intermediate): A combination of declarative configurations for complex policies and DSLs for simpler tasks. This approach helps balance ease of use and maintainability.
For Maturity Level 2 (Advanced): Declarative configurations can be used more extensively, with DSLs still being valuable for certain scenarios where scripting is required but not overly complex.
For Maturity Level 3 (Highly Skilled Developers): Code-based solutions are often the best choice. They provide maximum flexibility and control over the remediation logic.

#### Closing Summary

In summary, the right abstraction level depends on a combination of engineering maturity, policy complexity, maintainability needs, and deployment speed requirements. A thoughtful approach involves understanding these factors deeply to make an informed decision that maximizes efficiency and minimizes maintenance overhead.

#### Common Mistakes/Follow-Ups
Overlooking Engineering Maturity: Assuming all engineers have the same level of skill can lead to inefficiencies or frustration.
Ignoring Complexity: Failing to recognize that simple policies might require complex DSLs or declarative configurations can result in unnecessary workarounds.
Neglecting Deployment Speed: Rushing into code-based solutions without considering the time and effort required for development, testing, and deployment.

When designing remediation policies, always consider a phased approach where you start with simpler abstractions and gradually transition to more complex ones as needed. This ensures that all stakeholders are aligned and that the system remains maintainable over time.

**Comparison:**

```
Engineering Maturity Level	Recommended Abstraction
Beginner/Intermediate	DSLs, Declarative Configurations
Advanced	Declarative Configurations
Highly Skilled Developers	Code-Based Solutions
```

**Code:**

```
Example of a simple DSL policy definition in YAML

policy: action: "restart_service" condition: "service_status == 'down'"

CODE
Copy

```yaml
# Example of a declarative configuration for a complex policy
service_policy:
  - service_name: "webserver"
    actions:
      - type: "restart"
        when: "status_down"
      - type: "log_error"
        when: "retry_exceeded"
```

**Example input / output:**

```text
Input (Declarative Config):
{
  "service_policy": [
    {
      "service_name": "webserver",
      "actions": [
        {
          "type": "restart",
          "when": "status_down"
        },
        {
          "type": "log_error",
          "when": "retry_exceeded"
        }
      ]
    }
  ]
}

Output (Automated Action):
- Service webserver restarted due to status down.
- Error logged: Retry exceeded for service webserver.
```

**Data:**

```

{ "policy": { "action": "restart_service", "condition": "service_status == 'down'" } }

CODE
Copy

```yaml
service_policy:
  - service_name: "webserver"
    actions:
      - type: "restart"
        when: "status_down"
      - type: "log_error"
        when: "retry_exceeded"
```

**Flow:**

1. High
2. Low
3. Assess Engineering Maturity
4. Complexity?
5. DSLs & Declarative Config
6. Declarative Configurations
7. Code-Based Solutions forComplex Cases

