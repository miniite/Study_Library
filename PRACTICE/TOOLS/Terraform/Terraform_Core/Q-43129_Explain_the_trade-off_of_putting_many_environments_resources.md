---
type: "practice"
difficulty: "Deep"
---

# Explain the trade-off of putting many environments' resources under for_each in one state versus separate state per environment. When does each win?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring with State Moves](<../../../../TOOLS/Terraform/Terraform_Core/Refactoring_with_State_Moves.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When managing multiple environments with Terraform, you often have to decide whether to place all environments within a single state file or to use separate state files for each environment. This decision involves understanding how Terraform's for_each works and its implications on state management, resource versioning, and overall complexity.

#### Triage

The question is primarily about balancing the advantages and disadvantages of consolidating multiple environments into one Terraform state versus maintaining separate states for each environment. The trade-offs are centered around manageability, version control, isolation, and potential performance impacts.

Detailed Solution
Single State with for_each

When using a single state file, you can leverage Terraform’s for_each functionality to dynamically create resources across multiple environments. This approach is particularly useful when the number of environments or their configuration similarities are substantial. For example:

HCL
resource "aws_instance" "example" {
  for_each = { for env in var.environments : env.name => env }

  ami           = each.value.ami_id
  instance_type = each.value.type

  tags = {
    Name = "example-${each.key}"
  }
}

Here, var.environments is a list of objects containing details about each environment (e.g., name, AMI ID, type). This method allows you to define all environments in one place and dynamically provision instances based on their configuration.

Separate States per Environment

Using separate states for each environment offers clear isolation. Each state file is dedicated to a single environment, making it easier to manage and understand the specific configurations without worrying about cross-environment interference. For example:

HCL
# main.tf in env1/
resource "aws_instance" "example" {
  ami           = var.amis["dev"]
  instance_type = var.types["dev"]

  tags = {
    Name = "example-dev"
  }
}

# main.tf in env2/
resource "aws_instance" "example" {
  ami           = var.amis["prod"]
  instance_type = var.types["prod"]

  tags = {
    Name = "example-prod"
  }
}

Each environment has its own state file, and the configuration is defined within a single main.tf for simplicity.

#### Trade-offs

Manageability:

Single State: Easier to manage and understand overall infrastructure as it’s all in one place.
Separate States: Each environment is isolated, making it easier to track changes and revert configurations without affecting other environments.

Version Control:

Single State: Single version control for all environments can lead to complex state management if multiple teams are working on different environments simultaneously.
Separate States: Each team or environment has its own history, which simplifies tracking and debugging of changes.

Isolation:

Single State: Potential risk of accidental changes affecting other environments due to shared state.
Separate States: Strict isolation reduces the risk of configuration drift between environments.

Performance:

Single State: Can be more efficient in terms of state management and querying a single large state file.
Separate States: Smaller, dedicated states may perform better due to reduced complexity.

Complexity:

Single State: Dynamic for_each logic can add complexity in managing resource changes across environments.
Separate States: Simpler to manage as each environment has a straightforward configuration.

#### Conclusion

The decision between single and separate states depends on the specific needs of your organization. If you value simplicity, manageability, and dynamic provisioning capabilities, consolidating environments into one state with for_each might be more suitable. However, if you need strict isolation, clear version control, and simpler management of individual environments, maintaining separate states is a better choice.

#### Common Mistakes
Not considering the complexity: Overlooking the added complexity of managing multiple environments in a single state can lead to maintenance issues.
Ignoring versioning: Failing to manage state versioning properly in a single state file can result in hard-to-debug configuration drift.
Underestimating isolation: Assuming that separate states are too complex without understanding how Terraform’s data sources and outputs can help maintain isolation.

#### Closing Summary

In summary, the choice between using a single state with for_each or separate states for each environment depends on your organization's specific requirements. A thorough assessment of manageability, version control, isolation, and performance will guide you to the best approach.

**Comparison:**

```
Environment	AMI ID	Instance Type
dev	ami-1234567890abcdef0	t2.micro
prod	ami-0987654321fedcba9	m5.large
```

**Code:**

```hcl
# Example main.tf with for_each
resource "aws_instance" "example" {
  for_each = { for env in var.environments : env.name => env }

  ami           = each.value.ami_id
  instance_type = each.value.type

  tags = {
    Name = "example-${each.key}"
  }
}

# Example main.tf for separate states
resource "aws_instance" "example" {
  ami           = var.amis["dev"]
  instance_type = var.types["dev"]

  tags = {
    Name = "example-dev"
  }
}
```

**Example input / output:**

```text
Input:
- environments = [
    { name="dev", ami_id="ami-1234567890abcdef0", type="t2.micro" },
    { name="prod", ami_id="ami-0987654321fedcba9", type="m5.large" }
  ]

Output:
- In a single state, two AWS instances are created: one for dev and one for prod.
- Each instance is tagged with its respective environment name.
```

**Data:**

```json
{
  "environments": [
    { "name": "dev", "ami_id": "ami-1234567890abcdef0", "type": "t2.micro" },
    { "name": "prod", "ami_id": "ami-0987654321fedcba9", "type": "m5.large" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Single State with for_each
4. Does this simplifymanagement?
5. Use Single State
6. Maintain Separate States perEnvironment
7. Manage multipleenvironments dynamically
8. Ensure clear version controland isolation

