---
type: "practice"
difficulty: "Deep"
---

# How would you enforce that no VM in prod has a public IP and every storage account uses private endpoints, using Azure Policy with deny and remediation?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Key Vault and Secrets](<../../../../TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that no virtual machine (VM) in production uses a public IP and every storage account utilizes private endpoints, we can leverage Azure Policy to enforce these security measures. The approach involves creating two separate policies—Deny Public IPs on VMs in Prod and Ensure Storage Accounts Use Private Endpoints. Each policy will have deny rules and remediation steps defined to automatically correct any non-compliant resources.

**Comparison:**

```
Policy Name	Effect	Description
Deny Public IP VMs in Prod	deny	Prevent any VMs with public IPs from existing in the Production environment.
Ensure Storage Accounts Use Private Endpoints	deny	Enforce that all storage accounts must have private endpoints configured.
```

**Code:**

```

// Policy Definition for Denying Public IPs on VMs in Prod { "mode": "All", "parameters": {}, "policyRule": { "if": { "allOf": [ { "field": "type", "equals": "Microsoft.Compute/virtualMachines" }, { "field": "[parameters('env')].properties.environment", "equals": "Production" }, { "not": { "field": "Microsoft.Network/networkInterfaces[].ipConfigurations[].publicIPAddress.id", "exists": false } } ] }, "then": { "effect": "deny" } }, "metadata": { "category": "Security", "description": "Deny public IP on VMs in prod." } }

CODE
Copy

```json
// Policy Definition for Ensuring Storage Accounts Use Private Endpoints
{
  "mode": "All",
  "parameters": {},
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Storage/storageAccounts"
        },
        {
          "not": {
            "field": "[concat('Microsoft.Network/privateEndPoints/', parameters('storageName'))].id",
            "exists": true
          }
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  },
  "metadata": {
    "category": "Security",
    "description": "Ensure storage accounts use private endpoints."
  }
}
```

**Example input / output:**

```text
**Dry-run Input:**
- `virtualMachines` in the `Production` environment with public IPs.
- A `storageAccount` without a private endpoint.

**Expected Output:**
- The policy enforcement will deny any VMs in the Production environment that have public IPs and request remediation actions to remove them.
- For the storage account, it will deny access unless a private endpoint is attached and provide instructions on how to add one.
```

**Data:**

```json
{
  "environment": {
    "Production": {
      "publicIPDenyRule": true
    },
    "Development": {
      "publicIPDenyRule": false
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Initial Policy Creation
6. Are VMs in Prod using publicIPs?
7. Deny Public IP Policy
8. Check Storage Accounts forPrivate Endpoints
9. Do Storage Accounts haveprivate endpoints?
10. Deny and Remediate
11. Policy Enforcement Complete

