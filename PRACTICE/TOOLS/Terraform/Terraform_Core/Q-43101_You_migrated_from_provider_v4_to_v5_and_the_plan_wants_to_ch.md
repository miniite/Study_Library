---
type: "practice"
difficulty: "Deep"
---

# You migrated from provider v4 to v5 and the plan wants to change dozens of attributes. How do state schema upgrades and moved/replaced blocks help here?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring with State Moves](<../../../../TOOLS/Terraform/Terraform_Core/Refactoring_with_State_Moves.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a migration from Terraform Provider v4 to v5, encountering numerous attribute changes can indeed be daunting. However, with the right approach using state schema upgrades and moved/replaced blocks, you can manage these transitions effectively without manual intervention or data loss. The key lies in understanding how these features work together to ensure your infrastructure is correctly updated.

Firstly, let's break down what we're dealing with: the plan indicates that dozens of attributes are changing, which could lead to Terraform wanting to replace existing resources entirely. This is where state schema upgrades and moved/replaced blocks come into play. These tools help in maintaining the integrity of your infrastructure while making necessary updates.

**Comparison:**

```
Attribute	Old Value	New Value
AMI	ami-12345678	ami-abcd1234
Tags	Name = "old-name"	-
```

**Code:**

```hcl
# Example of a resource definition before migration
resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  tags = {
    Name = "old-name"
  }
}
```

**Example input / output:**

```text
# Plan output from Terraform v5 for migration
Terraform will perform the following actions:

- Replace existing aws_instance.example with new resource of type aws_instance

Changes to attributes:
  + ami           = "ami-abcd1234"
  - tags          = [null] (known data loss)
```

**Flow:**

1. No
2. Yes
3. Initiate Terraform Plan
4. Is the state schema up to date?
5. Run terraform statereplace-module
6. Verify resources are correctlyreplaced and attributesupdated
7. Continue with plan execution

