---
type: "practice"
difficulty: "Expert"
---

# What's your framework for deciding when data governance should be enforced as policy-as-code at the retrieval gateway vs. baked into each application?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Policy as Code for Data](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Policy_as_Code_for_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a data strategy and governance framework, the key decision revolves around whether to enforce data governance policies via code (policy-as-code) at the retrieval gateway or directly within each application. This choice depends on several factors related to scale, complexity, consistency, and maintainability. Let's break down this decision from first principles.

Firstly, let's define what we mean by policy-as-code: it involves encoding data governance rules as software artifacts that can be versioned, reviewed, tested, and deployed using standard DevOps practices. The retrieval gateway approach enforces these policies centrally at the entry point of all data access requests, while applying them directly within applications involves embedding these rules in each application’s codebase.

The primary considerations include:

Scale: Centralizing governance can reduce redundancy and simplify maintenance for large-scale systems with many microservices.

#### Cons
istency: Ensuring compliance across multiple applications is easier when policies are enforced at a central point.
Complexity: Complex, cross-cutting policies may be more manageable in a centralized gateway.
Application Ownership: Applications that have tight control over their data can benefit from local enforcement to maintain autonomy.

Now, let's delve into the step-by-step reasoning:

Assess Application Complexity and Independence:

If applications are tightly coupled with their specific data requirements, it might be more practical to enforce policies locally.
Conversely, if applications share a common set of services or APIs for accessing data, centralizing governance can ensure consistent enforcement.

Evaluate Governance Needs:

For simple, non-negotiable rules (e.g., data masking for sensitive fields), a centralized approach might suffice.
More complex, context-dependent policies (e.g., dynamic access controls based on user roles) may require embedding in applications.

#### Cons
ider Integration and Deployment:

Centralized governance can simplify integration with CI/CD pipelines, but it requires thorough testing to ensure all gateways are up-to-date.
Local enforcement complicates deployment workflows but ensures immediate application of policies without additional latency from gateway interactions.

Risk Management:

Centralizing governance reduces the risk of misconfigurations or inconsistencies across applications.
However, it introduces a single point of failure; if the central service goes down, all access stops.

Maintenance and Scalability:

A centralized approach can scale more easily with new applications, reducing boilerplate code.
Local enforcement requires maintenance in each application but may offer better long-term flexibility as teams evolve their implementations independently.

In summary, the decision hinges on a balance of these factors. Centralizing governance via policy-as-code at the retrieval gateway is generally beneficial for large-scale systems where consistency and maintainability are critical. However, local enforcement within applications can provide more granular control and quicker response times in smaller or highly independent microservices architectures.

#### Common mistakes
include:

Overcentralizing policies without considering application complexity.
Underestimating the maintenance burden of centralizing governance.
Ignoring the potential for single points of failure in centralized systems.

**Comparison:**

```
Decision Factor	Centralized (Gateway)	Local Enforcement
Scale	Easier to manage	More complex
Consistency	High	Variable
Complexity	Lower	Higher
Application Ownership	Central control	Local autonomy
```

**Code:**

```python
# Example of policy-as-code enforcement at retrieval gateway

def enforce_policy(data, user):
    # Implement logic to check and apply policies here
    if is_sensitive_data(data) and not has_access(user):
        mask_sensitive_fields(data)
    return data
```

**Example input / output:**

```text
Input:
- User: "Alice", Role: "Admin"
- Data: {"credit_score": 750, "social_security_number": "123-45-6789"}

Output:
- Masked Data: {"credit_score": 750, "social_security_number": "[MASKED]"}
```

**Flow:**

1. Yes
2. No
3. Application
4. Is the policy cross-cutting?
5. Enforce at Gateway
6. Enforce in App Codebase

