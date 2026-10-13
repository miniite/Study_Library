---
type: "practice"
difficulty: "Deep"
---

# How would you set up automated compliance reporting (CIS/PCI) on Azure using Defender for Cloud and Policy, and remediate at scale?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Key Vault and Secrets](<../../../../TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the setup of automated compliance reporting in Azure using Defender for Cloud and policies, we need to take a comprehensive approach that ensures both compliance monitoring and automatic remediation at scale. The core idea is to leverage Azure Policy for enforcing security standards such as CIS or PCI controls, and use Azure Security Center (Defender) for continuous assessment and reporting.

#### Triaging the Question

The question revolves around setting up an automated system for compliance monitoring in Azure using Azure Defender and Azure Policy. The key aspects being probed include:

Understanding of Azure Policies: How to define and manage policies that enforce security controls.
Azure Security Center (Defender): Utilizing its features to assess the compliance status and generate reports.
Remediation Capabilities: Automating the process of remediation based on detected non-compliant states.
Scalability Considerations: Ensuring the solution can handle large-scale environments without manual intervention.

#### First Principles Solution
Define Azure Policies: Create policies for CIS or PCI controls using the Azure Policy portal, PowerShell, or Azure CLI.
Configure Security Center (Defender): Enable and configure Security Center to use these policies.
Generate Compliance Reports: Set up scheduled reports in Security Center to monitor compliance status over time.
Automate Remediation: Implement custom scripts or Azure Automation runs to remediate non-compliant states.
Step-by-Step Solution

Define Policies:

Navigate to the Azure Policy portal and define policies based on CIS or PCI controls.

#### Example
policy for ensuring secure key management:
JSON
{
  "mode": "All",
  "parameters": {},
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.KeyVault/vaults"
        },
        {
          "not": {
            "field": "[concat('Microsoft.KeyVault/vaults/', parameters('name'))].properties.enableSoftDelete"
          }
        }
      ]
    },
    "then": {
      "effect": "audit"
    }
  },
  "metadata": {
    "category": "Security & Compliance",
    "description": "Audit Key Vault soft delete setting is enabled.",
    "version": "1.0"
  }
}

Enable Security Center (Defender):

Navigate to Azure Security Center and enable policies.
Configure the policy set definitions if needed, ensuring they cover all necessary controls.

Generate Compliance Reports:

Set up scheduled reports in Security Center to monitor compliance status over time.

#### Example
report configuration:
JSON
{
  "reportDefinition": {
    "title": "Compliance Report",
    "timeFrame": "Last 7 days",
    "filter": {}
  }
}

Automate Remediation:

Use Azure Automation to create and run scripts that remediate non-compliant states.

#### Example
PowerShell script for soft delete policy:
POWERSHELL
$resourceGroupName = 'your-resource-group-name'
$vaultName = 'your-key-vault-name'

$keyVault = Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $vaultName

if (-not $keyVault.EnableSoftDelete) {
    Set-AzKeyVault -EnableSoftDelete $true -ResourceGroupName $resourceGroupName -VaultName $vaultName
    Write-Output "Enabled soft delete for Key Vault."
} else {
    Write-Output "Soft delete is already enabled for Key Vault."
}

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key steps involve defining policies, enabling Security Center, generating reports, and automating remediation. Scalability can be managed by ensuring policies are correctly scoped, leveraging Azure's built-in automation features, and regularly reviewing and updating policies.

#### Common mistakes
include:

Failing to properly scope policies, leading to unnecessary alerts or misconfigurations.
Not configuring scheduled reports, resulting in delayed detection of compliance issues.
Overlooking the need for continuous review and updates to policies as environments evolve.

#### Follow-ups
could include discussing additional security controls, integrating with other Azure services like Security Center rules, or exploring advanced remediation techniques using custom runbooks.

**Comparison:**

```
Policy ID	Description
0987654321	Soft delete setting audit
1234567890	Network security group
Remediation Action	Status
Enable soft delete	Success
```

**Code:**

```powershell
# Example policy definition in JSON
$policy = @"
{
    "mode": "All",
    "parameters": {},
    "policyRule": {
        "if": {
            "allOf": [
                {
                    "field": "type",
                    "equals": "Microsoft.KeyVault/vaults"
                },
                {
                    "not": {
                        "field": "[concat('Microsoft.KeyVault/vaults/', parameters('name'))].properties.enableSoftDelete"
                    }
                }
            ]
        },
        "then": {
            "effect": "audit"
        }
    },
    "metadata": {
        "category": "Security & Compliance",
        "description": "Audit Key Vault soft delete setting is enabled.",
        "version": "1.0"
    }
}
"@

# Example PowerShell script for remediation
$resourceGroupName = 'your-resource-group-name'
$vaultName = 'your-key-vault-name'

$keyVault = Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $vaultName

if (-not $keyVault.EnableSoftDelete) {
    Set-AzKeyVault -EnableSoftDelete $true -ResourceGroupName $resourceGroupName -VaultName $vaultName
    Write-Output "Enabled soft delete for Key Vault."
} else {
    Write-Output "Soft delete is already enabled for Key Vault."
}
```

**Example input / output:**

```

Example input for setting up a policy:

POWERSHELL
Copy
Set-AzPolicyDefinition -Policy $policy -Name 'SoftDeleteSetting'

Output after remediation script execution:

CODE
Copy
Enabled soft delete for Key Vault.
```

**Data:**

```json
{
  "policyDefinition": "$policy",
  "scheduledReport": {
    "title": "Compliance Report",
    "timeFrame": "Last 7 days"
  }
}
```

**Flow:**

1. Define Azure Policies
2. Enable Security Center
3. Generate Compliance Reports
4. Automate Remediation
5. Monitor and Review

