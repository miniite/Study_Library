---
type: "practice"
difficulty: "Deep"
---

# You're setting the org's policy on where durable state may live. How do you make 'stateless by default, state consciously placed' an enforced platform property rather than a slogan, so ephemeral tiers never accumulate hidden state?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Stateless by Default](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce "stateless by default, state consciously placed" as a core principle in our cloud architecture, we need to systematically design and implement policies that ensure temporary or transient resources do not inadvertently store stateful data. This involves leveraging immutable infrastructure practices, resource tagging, automation, and security controls. Let's break down the approach from first principles.

First, understand that "stateless by default" means ephemeral tiers (like compute instances, session-based services, etc.) should not store any application-specific data in volatile storage like memory or disk. This ensures that if a new instance is spun up, it behaves identically to its predecessor without relying on persisted state.

To enforce this, we need to define and implement specific policies at multiple levels:

Resource Design: Ensure all temporary tiers are designed with ephemeral storage only for runtime data. Use local ephemeral disks (e.g., /tmp) where possible.
Tagging and Labeling: Implement a tagging strategy that identifies which resources hold stateful vs. stateless data. For example, tag stateful instances with stateful=true so they can be monitored separately from stateless ones.
Automation and Orchestration: Use CI/CD pipelines to deploy applications in a way that minimizes the need for stateful storage. This includes using secrets management tools (like AWS Secrets Manager) and ensuring sensitive data is not stored locally but rather fetched at runtime via secure means.
Monitoring and Alerting: Set up monitoring to detect any unauthorized persistence of state on ephemeral resources. Use tools like CloudWatch Alarms or Stackdriver for proactive alerting.
Security Policies: Enforce strict security policies that prohibit writing to persistent storage from transient resources. This can be achieved through network segmentation, role-based access control (RBAC), and policy enforcement via cloud provider-native mechanisms.

By focusing on these areas, we can make "stateless by default" an enforced platform property rather than just a slogan.

**Comparison:**

```
Resource Type	State	Storage Type	Size
Web Stateless	False	Ephemeral	10GB
Web Stateful	True	Persistent	50GB
```

**Code:**

```python
# Example of tagging resources in Terraform for stateful vs. stateless instances
resource "aws_instance" "stateless" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  # Tagging the instance as stateless by default
  tags = {
    Name      = "web-stateless"
    Stateful  = "false"
  }
}

resource "aws_instance" "stateful" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  # Tagging the instance as stateful
  tags = {
    Name      = "web-stateful"
    Stateful  = "true"
  }
}
```

**Example input / output:**

```text
Input:
- Deploy a new web application using Terraform.

Output:
- The Terraform script tags the instances appropriately and sets up ephemeral storage for stateless instances, while stateful instances have persistent storage.
```

**Data:**

```json
{
  "resources": [
    {
      "name": "web-stateless",
      "tags": {
        "Name": "web-stateless",
        "Stateful": "false"
      },
      "storage": [
        { "type": "ephemeral", "size": "10GB" }
      ]
    },
    {
      "name": "web-stateful",
      "tags": {
        "Name": "web-stateful",
        "Stateful": "true"
      },
      "storage": [
        { "type": "persistent", "size": "50GB" }
      ]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Resource Creation
6. Is the resource tagged asstateless?
7. Create ephemeral storage
8. Does the resource requirepersistent storage?
9. Create persistent storage
10. Do not create storage, proceedwith deployment

