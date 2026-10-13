---
type: "practice"
difficulty: "Deep"
---

# How would you make an Azure Storage account more secure (disable public access, use private endpoints, rotate keys)?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure Key Vault and Secrets](<../../../../TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enhance the security of an Azure Storage account, we need to focus on several key areas: disabling public access, using private endpoints, and rotating storage account keys. These steps ensure that your data is protected from unauthorized access and potential threats.

#### Triage
The question is asking how to make an Azure Storage account more secure by implementing best practices such as disabling public access, setting up private endpoints for network isolation, and regularly rotating the storage account keys. The interviewer is probing your understanding of security principles in Azure and your ability to implement them effectively.

#### Assumptions

We are working with a standard Azure Storage account.
The organization has basic knowledge of Azure resources and permissions.
There is an existing storage account that needs to be secured.

#### First-principles solution

Disable Public Access:

Public access in Azure Storage allows anyone on the internet to read or write data without needing authentication. To disable public access, follow these steps:
POWERSHELL
# Enable private access for the storage account
$storageAccount = Get-AzStorageAccount -ResourceGroupName "your-resource-group" -Name "your-storage-account"
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName -Name $storageAccount.Name -EnablePublicAccess False
This command sets the EnablePublicAccess property to False, effectively disabling public access.

Use Private Endpoints:

A private endpoint allows you to connect your storage account directly to a virtual network (VNet), isolating it from the internet and reducing exposure to potential threats.
POWERSHELL
# Create a private endpoint for the storage account
$vnet = Get-AzVirtualNetwork -ResourceGroupName "your-resource-group" -Name "your-virtual-network"
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "your-subnet"
New-AzPrivateEndpoint -ResourceGroupName "your-resource-group" -Name "your-private-endpoint" -PrivateLinkServiceId (Get-AzPrivateLinkServiceConnection -Name "storageAccountPrivateLinkService") -Subnet $subnet
After creating the private endpoint, update the storage account network rules to use this VNet:
POWERSHELL
# Update the storage account's network rules using the private link
Set-AzStorageAccountNetworkRuleSet -ResourceGroupName "your-resource-group" -Name "your-storage-account" -DefaultAction Allow -VirtualNetwork $vnet -Subnets @($subnet.Id)

Rotate Storage Account Keys:

Regularly rotating storage account keys ensures that even if a key is compromised, the damage can be minimized.
POWERSHELL
# Rotate the primary and secondary keys for the storage account
$storageAccount = Get-AzStorageAccount -ResourceGroupName "your-resource-group" -Name "your-storage-account"
$newKeys = New-AzStorageContext -ConnectionString (Get-AzStorageAccountKey -ResourceGroupName $storageAccount.ResourceGroupName -Name $storageAccount.Name).Value[0]
# Update any code or scripts using the old keys to use the new ones

#### Closing summary
By following these steps, you can significantly enhance the security of your Azure Storage account by disabling public access, setting up private endpoints for network isolation, and regularly rotating storage account keys. These measures help protect sensitive data from unauthorized access and potential threats.

#### Common mistakes/follow-ups

Failing to understand that private endpoints require a VNet setup.
Not updating all relevant code or scripts after key rotation.
Overlooking the importance of monitoring and auditing for security breaches.

