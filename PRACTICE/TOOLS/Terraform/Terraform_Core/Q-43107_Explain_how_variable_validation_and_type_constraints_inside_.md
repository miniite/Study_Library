---
type: "practice"
difficulty: "Intermediate"
---

# Explain how variable validation and type constraints inside a module protect callers. Give an example where they'd catch a bad input before apply.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Module Compatibility](<../../../../TOOLS/Terraform/Terraform_Core/Module_Compatibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When working with Terraform, using modules can encapsulate complex resources and configurations into reusable components. However, to ensure that these modules are used correctly by callers, we need mechanisms to validate inputs and enforce type constraints. This helps prevent errors and ensures that the infrastructure deployments follow a consistent and reliable pattern.

#### Triaging the Question

The question is asking us to explain how variable validation and type constraints inside a Terraform module protect the callers of the module. Specifically, it wants an example where these mechanisms catch bad input before the terraform apply command executes.

#### First-Principles Solution
Module Structure: A Terraform module typically has a variables.tf file where you define input variables and their constraints.
Validation Logic: Inside your module's code, there are validation checks that run during the planning phase to ensure inputs meet specified criteria.
Error Handling: If the validation fails, Terraform will return an error before applying any changes.

#### Example

Let’s say we have a networking module that deploys VPCs and subnets in AWS. The module requires parameters like region, network CIDR block, and subnet sizes.

Step 1: Defining Variables with Constraints

In the variables.tf file of the module:

HCL
variable "region" {
  description = "The AWS region where resources will be created."
  type        = string
  validation {
    condition      = length(var.region) > 0
    error_message  = "Region must not be empty."
  }
}

variable "vpc_cidr_block" {
  description = "The CIDR block for the VPC."
  type        = string
  validation {
    condition      = cidrsubnet(var.vpc_cidr_block, 24, 0) > 16384
    error_message  = "VPC CIDR block must be within a valid range (not overlapping with private IPs)."
  }
}

variable "subnet_count" {
  description = "Number of subnets to create."
  type        = number
  validation {
    condition      = var.subnet_count > 0 && var.subnet_count <= 4
    error_message  = "Must have between 1 and 4 subnets."
  }
}
Step 2: Validation in Module Code

Inside the module's main.tf file, we can include checks to ensure that the VPC CIDR block is valid for subnetting:

HCL
resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr_block

  tags = {
    Name = "example-vpc"
  }
}

locals {
  subnets = [for i in range(var.subnet_count) : "${var.vpc_cidr_block}-${(i * 16 + 2)}"
             if cidrsubnet(var.vpc_cidr_block, 24, (i * 16 + 2)) != "0.0.0.0/255.255.255.255"]
}

resource "aws_subnet" "this" {
  count = length(local.subnets)

  cidr_block        = element(local.subnets, count.index)
  vpc_id            = aws_vpc.this.id
  availability_zone = var.availability_zones[count.index % length(var.availability_zones)]
}
Step 3: Running terraform plan

When a caller of the module tries to apply this with invalid inputs, Terraform will fail validation and return an error message:

SH
$ terraform apply -var-file=network.tfvars

Error: Invalid value for argument

  on main.tf line 4, in resource "aws_vpc" "this":
   4:   cidr_block = var.vpc_cidr_block

The given CIDR block is not a valid IPv4 CIDR block. Check if the input is correct.

#### Closing Summary

Variable validation and type constraints in Terraform modules ensure that only valid inputs are passed to the module’s resources, which prevents configuration drift and errors during deployment.

#### Common Mistakes
and Follow-Ups
Mistake: Not defining validation conditions for all variables, leading to unexpected behavior.

#### Follow-Up
Discussing how to handle dynamic or complex variable types (e.g., maps, lists).
Mistake: Overlooking the importance of error messages in guiding callers towards correct usage.

#### Follow-Up
Exploring advanced validation techniques like custom functions and external validation tools.

**Comparison:**

| Variable     | Type       | Validation Conditions                   | Error Message                                           |
|--------------|------------|-----------------------------------------|--------------------------------------------------------|
| region       | string     | length(var.region) > 0                  | Region must not be empty.                               |
| vpc_cidr_block | string    | cidrsubnet(var.vpc_cidr_block, 24, 0) > 16384 | VPC CIDR block must be within a valid range (not overlapping with private IPs). |
| subnet_count | number     | var.subnet_count > 0 && var.subnet_count <= 4 | Must have between 1 and 4 subnets.                     |

**Code:**

```hcl
# variables.tf
variable "region" {
  description = "The AWS region where resources will be created."
  type        = string
  validation {
    condition      = length(var.region) > 0
    error_message  = "Region must not be empty."
  }
}

variable "vpc_cidr_block" {
  description = "The CIDR block for the VPC."
  type        = string
  validation {
    condition      = cidrsubnet(var.vpc_cidr_block, 24, 0) > 16384
    error_message  = "VPC CIDR block must be within a valid range (not overlapping with private IPs)."
  }
}

variable "subnet_count" {
  description = "Number of subnets to create."
  type        = number
  validation {
    condition      = var.subnet_count > 0 && var.subnet_count <= 4
    error_message  = "Must have between 1 and 4 subnets."
  }
}

# main.tf
resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr_block

  tags = {
    Name = "example-vpc"
  }
}

locals {
  subnets = [for i in range(var.subnet_count) : "${var.vpc_cidr_block}-${(i * 16 + 2)}"
             if cidrsubnet(var.vpc_cidr_block, 24, (i * 16 + 2)) != "0.0.0.0/255.255.255.255"]
}

resource "aws_subnet" "this" {
  count = length(local.subnets)

  cidr_block        = element(local.subnets, count.index)
  vpc_id            = aws_vpc.this.id
  availability_zone = var.availability_zones[count.index % length(var.availability_zones)]
}
```

**Example input / output:**

```text
$ terraform apply -var-file=network.tfvars

Error: Invalid value for argument

  on main.tf line 4, in resource "aws_vpc" "this":
   4:   cidr_block = var.vpc_cidr_block

The given CIDR block is not a valid IPv4 CIDR block. Check if the input is correct.
```

**Data:**

```json
# network.tfvars
region = "us-west-2"
vpc_cidr_block = "10.0.0.0/16"
subnet_count = 3
availability_zones = ["us-west-2a", "us-west-2b", "us-west-2c"]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is input valid?
5. Apply changes
6. Display error message andterminate
7. End
8. End

