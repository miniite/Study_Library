---
type: "practice"
difficulty: "Deep"
---

# Where would you store Terraform state for Azure and why is a storage account backend common?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure SQL and App Service Deployments](<../../../../TOOLS/Azure/Azure_Platform/Azure_SQL_and_App_Service_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store Terraform state in Azure, a storage account backend like Azure Blob Storage or Azure Table Storage is commonly used. This approach ensures that your infrastructure as code (IaC) changes are versioned and stored securely. Let's break down why this is the case from first principles.

**Code:**

```plaintext
# Example Terraform configuration using Azure Blob Storage
resource "azurerm_storage_account" "example" {
  name                = "tfstatestorage"
  resource_group_name = "terraform-resources"
  location            = "eastus"
  account_tier        = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_container_registry" "example" {
  # Other configurations
}

resource "null_resource" "set_terraform_state_backend" {
  triggers = {
    backend_config = jsonencode({
      config = {
        backend = "azurerm",
        bucket = azurerm_storage_account.example.blob_endpoint,
        key    = "terraform.tfstate",
        region = azurerm_storage_account.example.primary_location
      }
    })
  }

  provisioner "local-exec" {
    command = "echo 'backend \"azurerm\" { bucket = \${azurerm_storage_account.example.name} key = \"terraform.tfstate\" region = \${azurerm_storage_account.example.location} }' > .terraform/terraform.tfstate.d/azure-backend.hcl"
  }
}
```

**Flow:**

1. Initialize Storage Account
2. (Create Storage AccountResource)
3. Configure Backend forTerraform
4. Store Terraform State in AzureBlob/Table Storage

