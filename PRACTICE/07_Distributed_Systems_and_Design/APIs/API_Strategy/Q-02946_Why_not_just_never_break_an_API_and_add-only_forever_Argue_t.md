---
type: "practice"
difficulty: "Intermediate"
---

# Why not just never break an API and add-only forever? Argue the failure modes of an add-only-forever policy at 10-year horizon — schema bloat, semantic drift, security debt — and where you'd draw the line and force a break.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Versioning and Deprecation Governance](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Versioning_and_Deprecation_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an API strategy, one might consider an "add-only-forever" policy to ensure backward compatibility and avoid breaking existing clients. However, such an approach can lead to significant issues over time, particularly in a 10-year horizon. The core challenges include schema bloat, semantic drift, and security vulnerabilities. These issues can accumulate to the point where maintaining the API becomes unsustainable or poses unacceptable risks.

Let's dive into each of these failure modes and discuss when you might need to break an API to address them effectively.

Schema Bloat

Over time, APIs tend to grow in complexity due to the addition of new features and data requirements. An "add-only-forever" policy can lead to a situation where the API becomes bloated with unnecessary fields, methods, and endpoints that are no longer relevant or useful. For example, consider an e-commerce API that initially only required basic product information but later added detailed reviews, customer ratings, and inventory levels. While these additions provide valuable functionality, they could introduce complexity and performance issues for clients that don’t need the full suite of features.

Semantic Drift

Semantic drift occurs when the meaning or interpretation of data changes over time. This can happen due to evolving business needs, changing regulations, or shifts in how certain data is used within an organization. For instance, consider a financial API that initially defined "price" as a simple monetary value but later began to include additional metadata such as currency type and validity period. These changes might not be immediately apparent to clients, leading to confusion and potential errors.

Security Debt

Security threats evolve over time, and APIs must adapt to address new vulnerabilities. An "add-only-forever" policy can accumulate security issues that become harder to manage as the API grows. For example, a legacy API might lack proper authentication mechanisms or data encryption standards that are later deemed inadequate by modern security practices. Over-reliance on an aging protocol could expose clients to significant risks, making it necessary to break the API and introduce new security measures.

Drawing the Line

Given these failure modes, you need to determine when and where to draw the line in forcing a breaking change. Here are some guidelines:

Regular Audits: Conduct regular reviews of the API’s schema and functionality to identify areas that no longer serve their intended purpose or have become overly complex.
Client Feedback: Engage with clients to understand their needs and pain points. This can help you prioritize which features should be deprecated or replaced.
Risk Assessment: Evaluate the potential impact of breaking changes on both internal and external stakeholders. Consider the cost-benefit ratio before making a decision.
Migration Strategies: Plan for smooth transitions by providing clear deprecation timelines, migration guides, and fallback mechanisms.

#### Example Scenario

Imagine an API that handles user authentication. Over time, it evolves to support multiple authentication methods (e.g., basic auth, OAuth2, JWT). However, the initial implementation of basic auth might have been flawed due to poor security practices. At some point, these flaws become too significant, and it is necessary to break the API and introduce a more secure approach.

#### Conclusion

While an "add-only-forever" policy seems appealing for maintaining backward compatibility, it can lead to schema bloat, semantic drift, and security debt over time. It’s crucial to periodically reassess the API and make breaking changes when necessary to ensure its long-term viability and security. The key is to balance continuous improvement with strategic disruptions that address critical issues.

#### Common mistakes
include ignoring these failure modes entirely or waiting too long before making a breaking change, which can complicate migrations and disrupt client workflows.

**Comparison:**

```
Schema	Current State	Ideal State
Users	Basic Fields	Enhanced Security & Features
Logs	Simple Entries	Detailed Audit Trails
```

**Code:**

```python
# Example of a simple function to demonstrate how schema changes might be handled
def update_user_data(user):
    if "new_field" in user:
        # Handle new field logic here
        pass
    else:
        # Fallback for old clients
        pass
```

**Example input / output:**

```
Input:
JSON
Copy
{
  "user": {
    "id": 123,
    "name": "John Doe",
    "email": "john.doe@example.com",
    "new_field": "some_value"
  }
}
Output:
JSON
Copy
{
  "user": {
    "id": 123,
    "name": "John Doe",
    "email": "john.doe@example.com",
    "new_field": "some_value"
  },
  "status": "success"
}
Dry Run:
PYTHON
Copy
user = {
  "id": 123,
  "name": "John Doe",
  "email": "john.doe@example.com",
  "new_field": "some_value"
}

def update_user_data(user):
    if "new_field" in user:
        # Handle new field logic here
        pass
    else:
        # Fallback for old clients
        pass

update_user_data(user)
```

**Data:**

```json
{
  "user": {
    "id": 123,
    "name": "John Doe",
    "email": "john.doe@example.com"
  }
}
```

**Flow:**

1. Yes
2. No
3. Identify Issues
4. Is Breaking Change Needed?
5. Migrate API
6. Routine Audits & FeedbackLoop
7. Smooth Transition Plan
8. Implement Changes

