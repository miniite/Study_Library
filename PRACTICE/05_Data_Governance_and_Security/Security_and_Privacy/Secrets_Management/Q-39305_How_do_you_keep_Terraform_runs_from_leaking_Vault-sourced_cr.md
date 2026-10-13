---
type: "practice"
difficulty: "Deep"
---

# How do you keep Terraform runs from leaking Vault-sourced credentials into state files across dozens of pipelines? Design the pattern and explain what still ends up in state regardless.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Bootstrapping Vault](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Bootstrapping_Vault.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of keeping Terraform runs from leaking Vault-sourced credentials into state files across multiple pipelines, we need to design a robust secrets management strategy that ensures sensitive data is never directly stored or leaked in state files. The key idea here is to use a combination of environment-specific variables and secure vault retrieval mechanisms.

The core problem is that Terraform state files can inadvertently include sensitive data if not properly managed. Specifically, this can happen when Vault credentials are stored directly within Terraform configurations or pipeline scripts. Our solution will leverage Vault's capabilities for dynamic secrets and environment-specific variable management to ensure that secrets are only retrieved at runtime.

Firstly, we need to understand the underlying issue: When using file backend for Terraform state files, any sensitive data required by a Terraform configuration must be hardcoded if not managed externally. This can lead to credentials being directly embedded in the state file or pipeline scripts. We will design a pattern that minimizes this risk.

#### Triage

The question is probing into how to securely manage and retrieve secrets in Terraform pipelines, ensuring that sensitive data does not inadvertently leak into state files. Key assumptions include:

Multiple pipelines are involved.
Vault is used as the secret management solution.
The goal is to avoid storing credentials directly within Terraform configurations or state files.

#### First-Principles Solution
Environment-Specific Variables: Define environment-specific variables in a separate file that is sourced at runtime by the pipeline. This ensures that sensitive data, like Vault tokens and paths, are not hard-coded.
Vault Integration with Terraform: Use Vault's dynamic secrets engine to generate credentials on-demand for each pipeline run. Store these credentials temporarily in the environment or as a part of the run context.
Terraform Configuration Management: Ensure that Terraform configurations are parameterized using variables sourced from secure vaults.
Step-by-Step Solution

Define Environment-Specific Variables:

Create separate .tfvars files for each environment (e.g., dev.tfvars, prod.tfvars). These files should contain sensitive data and be stored securely.

Vault Dynamic Secrets:

Configure Vault to provide dynamic secrets via the kv or approle engines.
Use Terraform's dynamic secret providers to retrieve these secrets at runtime.

Terraform Configuration:

Parameterize your Terraform configurations using variables from the .tfvars files.
Ensure that sensitive values are not stored in state files by leveraging Vault's dynamic secrets mechanism.

Pipeline Script:

Use a script to source environment-specific variables and pass them as arguments to Terraform commands.

#### Example
command: terraform apply -var-file=dev.tfvars.

#### Closing Summary

By using this pattern, we ensure that sensitive data is never stored directly in state files or pipeline scripts. Instead, it is dynamically retrieved at runtime from Vault. This approach minimizes the risk of credentials being inadvertently leaked.

#### Common Mistakes
and Follow-Ups
Mistake: Storing secrets in plain text within Terraform configurations.

#### Follow-Up
Consider using a centralized secret management solution like HashiCorp Vault to manage and retrieve secrets securely.
Environment-Specific Variables
Terraform Configuration
Vault Dynamic Secrets
Terraform Apply Commandwith -var-file

**Comparison:**

```
Variable	Description
VAULT_TOKEN	Vault token for authentication
TF_VAR_vault_token	Terraform variable to pass the vault token
```

**Code:**

```
Example Terraform variable file (dev.tfvars)

vault_token = "your-vault-token" vault_path = "secret/data/terraform"

CODE
Copy
```sh
# Example pipeline script snippet
export VAULT_TOKEN=$(cat /path/to/vault-token)
terraform apply -var-file=dev.tfvars
```

**Example input / output:**

```
Input: Pipeline script sourcing environment variables and invoking Terraform apply

export VAULT_TOKEN=$(cat /path/to/vault-token) export TF_VAR_vault_token=$VAULT_TOKEN

terraform apply -var-file=dev.tfvars

CODE
Copy
```plaintext
# Output: Running Terraform with dynamic secrets from Vault
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Changes to attributes of existing resources:
  + module.vpc = {
      + id = "vpc-123456"
    }
```

**Data:**

```text
- **Sample Vault Path**: `secret/data/terraform`
- **Terraform Variable File**: `dev.tfvars`, `prod.tfvars`
```

