---
type: "practice"
difficulty: "Deep"
---

# How do sensitive outputs and the sensitive flag behave in state versus plan output? Does marking sensitive actually protect the value at rest?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring with State Moves](<../../../../TOOLS/Terraform/Terraform_Core/Refactoring_with_State_Moves.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Terraform, understanding how sensitive behaves is crucial for managing secrets and other sensitive data. In this context, we need to distinguish between how Terraform handles sensitive outputs in the state versus plan output. Let's explore these concepts step-by-step from first principles.

#### Triage
What’s Being Probed

The question is probing your understanding of:

State vs Plan Outputs: How Terraform treats and displays sensitive data during different phases.
Security Implications: Whether marking a value as sensitive truly protects it at rest.
Key Concepts
Terraform State: The persistent storage where Terraform keeps the state of your infrastructure, which includes outputs, variables, resources, etc.
Plan Output: A temporary output generated during a Terraform run (e.g., terraform plan) showing what changes would be applied if you ran apply.
Sensitive Flag: Used to indicate that a value should not be logged or displayed in plain text.
Detailed Solution
Sensitive Outputs in State

When marking an output as sensitive, the sensitive data is stored in the Terraform state file. However, this does not mean the data is encrypted at rest; it merely means Terraform will mask the value when displaying it. For example:

HCL
output "sensitive_output" {
  description = "Sensitive information"
  value       = var.sensitive_info
  sensitive   = true
}

In the state file, var.sensitive_info might be stored in plain text, but Terraform will mask its display when you use commands like:

SH
terraform state list
terraform output -json
Sensitive Outputs in Plan Output

The behavior is similar during plan output. When you run terraform plan, the sensitive data will not be logged or displayed in plain text. For example, if you have a plan command that includes sensitive outputs:

SH
$ terraform plan
Plan: 0 to add, 0 to change, 0 to destroy.
output "sensitive_output" (sensitive value hidden)

Even though the data is not logged or displayed directly, it’s important to note that the sensitive values are processed by Terraform. They might be temporarily stored in memory or other intermediate locations where they could potentially be accessed if proper precautions aren’t taken.

Security Implications

Marking a value as sensitive does provide some level of protection because:

Masked Outputs: The values won't be logged to the console, reducing exposure during planning and debugging.
Configuration Best Practices: It encourages developers to treat sensitive data with care and not hard-code them directly in code or state files.

However, it doesn’t guarantee that the value is secure at rest. To ensure true security:

State Encryption: Terraform allows you to encrypt the state file using a backend like S3 or Vault.
Environment Variables: Use environment variables for sensitive data instead of hardcoding them in configuration files.
Secure Backends: Store your sensitive data in secure backends that are managed outside of your codebase.
Edge Cases and Best Practices
Plan vs Apply Outputs: Plan outputs mask sensitive values, but apply outputs (when using -out flag) might still log these values if not properly handled.
State File Security: Ensure the state file is stored securely, ideally encrypted at rest with strong access controls.
Audit Trails: Regularly audit your infrastructure and configuration to ensure compliance and security best practices are being followed.

#### Closing Summary

In summary, marking a value as sensitive in Terraform helps prevent accidental exposure of sensitive data during planning and debugging phases. However, it does not fully protect the value at rest. To achieve true protection, you need to combine this with secure backend storage and other best practices.

#### Common Mistakes/Follow-Ups
Overreliance on Sensitivity Flag: Failing to properly encrypt state files or store sensitive data securely.
Improper Handling of Sensitive Outputs in Apply: Not masking values when using apply outputs, leading to potential exposure.
Lack of Regular Audits: Ignoring regular checks for compliance and security best practices.

**Code:**

```hcl
output "sensitive_output" {
  description = "Sensitive information"
  value       = var.sensitive_info
  sensitive   = true
}
```

