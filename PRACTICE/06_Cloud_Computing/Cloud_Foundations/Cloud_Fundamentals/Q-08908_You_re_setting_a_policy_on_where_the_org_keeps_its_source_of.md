---
type: "practice"
difficulty: "Deep"
---

# You're setting a policy on where the org keeps its 'source of truth for infrastructure state.' How do you decide between provider-native, Terraform state, and a control-plane approach, and what breaks each?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Lifecycle and Retention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Data_Lifecycle_and_Retention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding where to keep the 'source of truth for infrastructure state' in an organization, we need to consider several factors including the nature of the infrastructure, the scale, the complexity, and the broader ecosystem. The three options—provider-native (e.g., AWS CloudFormation StackSets), Terraform state, and a control-plane approach (like HashiCorp Consul)—each have their strengths and weaknesses.

#### Triage

The question is essentially probing how we balance between leveraging cloud provider capabilities, using an infrastructure-as-code tool like Terraform, or implementing a more generic control-plane solution. We need to consider the trade-offs in terms of ease of management, security, governance, and resilience.

#### First-Principles Solution

Provider-Native Solutions (e.g., AWS CloudFormation StackSets):

#### Pros
Tight integration with cloud provider services.
Managed by the cloud provider, reducing operational overhead.
Versioning and change history built-in.

#### Cons
Limited flexibility compared to more general IaC tools.
Vendor lock-in.
Less control over state management.

Terraform State:

#### Pros
Highly flexible and extensible.
Widely supported by a large community, with numerous plugins and integrations.
Strong versioning capabilities and detailed change tracking.

#### Cons
Requires ongoing maintenance and governance policies.
Can be complex to manage at scale.

Control-Plane Approaches (e.g., HashiCorp Consul):

#### Pros
Decoupled from specific cloud providers, offering more flexibility.
Strong support for advanced features like service discovery and configuration management.
Holistic approach to state management across different systems and services.

#### Cons
Increased complexity due to additional infrastructure required.
Steeper learning curve.
Detailed Analysis

Provider-Native Solutions:

For simpler, less complex environments or when tight integration with cloud provider services is critical, provider-native solutions like AWS CloudFormation StackSets can be a good choice. They are managed by the cloud provider and require minimal ongoing maintenance.
However, they may not offer the same level of flexibility as Terraform, making them unsuitable for more dynamic or cross-cloud environments.

Terraform State:

For organizations with complex infrastructure needs and a desire to maintain full control over their state management process, Terraform is an excellent choice.
It allows for detailed versioning and change tracking, which can be crucial for compliance and governance requirements. However, it requires strong internal processes to ensure that the state file is managed correctly.

Control-Plane Approaches:

For large-scale, multi-cloud environments where a more holistic approach to state management is needed, control-plane solutions like HashiCorp Consul can be beneficial.
They provide advanced features such as service discovery and configuration management, which can simplify complex infrastructure setups.
However, the added complexity of managing additional infrastructure can be challenging.

#### Common Mistakes/Follow-Ups
Vendor Lock-In: Choosing a provider-native solution might lead to vendor lock-in, making it harder to switch cloud providers in the future.
Overcomplication: Using Terraform for simple environments or control-plane approaches for overly complex ones can result in unnecessary complexity and operational overhead.
Security and Governance: Ensuring proper security measures are in place when using state files or control planes is crucial but often overlooked.

In summary, the choice between these options depends on the specific needs of the organization. Provider-native solutions offer simplicity and integration but may lack flexibility, while Terraform provides great flexibility at the cost of ongoing maintenance. Control-plane approaches offer more advanced features but add complexity in terms of additional infrastructure and governance requirements.

**Comparison:**

```
Provider-Native	Terraform State	Control-Plane
Tight integration with cloud provider services.	Highly flexible and extensible.	Decoupled from specific cloud providers, offering more flexibility.
Managed by the cloud provider, reducing operational overhead.	Requires ongoing maintenance and governance policies.	Strong support for advanced features like service discovery and configuration management.
Versioning and change history built-in.	Holistic approach to state management across different systems and services.	Increased complexity due to additional infrastructure required.
```

**Code:**

```python
# Example of a simple Terraform state file structure

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```

**Example input / output:**

```text
# Example Terraform output for an AWS instance creation
output "instance_id" {
  value = aws_instance.web.id
}
```

**Data:**

```json
{
  "terraform_state": [
    {
      "type": "aws_instance",
      "name": "web",
      "attributes": {
        "ami": "ami-0c55b159cbfafe1f0",
        "instance_type": "t2.micro"
      }
    }
  ]
}
```

**Flow:**

1. Provider-Native Solutions
2. Tight Integration
3. Managed by Cloud Provider
4. Less Flexibility
5. Terraform State
6. Highest Flexibility
7. Community Support
8. Advanced Versioning
9. Control-Plane Approaches
10. Decoupled from Providers
11. Strong Advanced Features
12. Increased Complexity

