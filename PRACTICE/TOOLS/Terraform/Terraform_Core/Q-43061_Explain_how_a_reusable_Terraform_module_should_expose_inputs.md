---
type: "practice"
difficulty: "Intermediate"
---

# Explain how a reusable Terraform module should expose inputs and outputs. What makes a module hard to reuse across teams?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Module Compatibility](<../../../../TOOLS/Terraform/Terraform_Core/Module_Compatibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this answer, we will explore how to design reusable Terraform modules by properly exposing inputs and outputs. We'll also discuss common pitfalls that make modules difficult to reuse across different teams.

#### Triaging the Question

The core of the question revolves around creating well-designed Terraform modules that can be effectively reused within a team or across multiple projects. The key aspects include:

Inputs: How to define inputs and ensure they are flexible and generic enough.
Outputs: How to expose outputs so that downstream resources or teams can consume them easily.
Reusability: Factors contributing to the difficulty of reusing a module, such as hard-coded values and lack of abstraction.

#### First Principles Solution
Inputs

Parameterization with Variables: Define variables in your Terraform configuration files to accept input parameters. This allows users to pass their own values when using the module.

HCL
variable "instance_type" {
  description = "Instance type for EC2 instances"
  type        = string
  default     = "t2.micro"
}

Default Values: Provide sensible defaults for frequently used variables, but make sure these can be overridden.

Dynamic Variables: Use Terraform's for_each and map to support dynamic inputs.

Outputs

Defined Outputs: Explicitly define outputs in the module’s configuration file. This makes it clear what information the module will provide.

HCL
output "public_ip" {
  value = aws_instance.example.public_ip
}

Type Consistency: Ensure that the type of each output matches expected usage, such as strings for names and maps for configurations.

Reusability Challenges
Hardcoded Values: Avoid hardcoding values like region, VPC ID, or instance types directly in your module. These should be passed via inputs.
Environment-Specific Logic: Keep logic that depends on the environment (e.g., staging vs production) outside of the core logic and handle it externally.
Logging and Error Handling: Implement consistent logging and error handling to make debugging easier when using the module in different contexts.

#### Closing Summary

A reusable Terraform module should expose its inputs through variables, allowing flexibility, and define clear outputs for downstream consumption. To ensure reusability across teams, avoid hardcoding values, keep environment-specific logic separate, and maintain consistent logging practices.

#### Common Mistakes/Follow-Ups
Hardcoded Values: Failing to parameterize crucial pieces of information.
Overly Complex Outputs: Exposing too much or poorly structured output can complicate downstream processes.
Lack of Documentation: Not providing clear documentation on how to use the module, including examples and best practices.
Depth + Language

This explanation is tailored for an intermediate-level engineer familiar with Terraform basics but needing a refresher on best practices for module creation and reuse.

