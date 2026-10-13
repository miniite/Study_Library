---
type: "practice"
difficulty: "Intermediate"
---

# What is a SAS token in Azure Storage and how is it different from a storage account key?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure Key Vault and Secrets](<../../../../TOOLS/Azure/Azure_Platform/Azure_Key_Vault_and_Secrets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A SAS (Shared Access Signature) token in Azure Storage allows you to grant limited access to specific resources within a storage account, without sharing your full storage account key. This method is more secure than exposing your storage keys because it restricts the permissions and duration of access. The main difference between a SAS token and a storage account key lies in their usage: while storage account keys are used for authenticating the entire storage account, SAS tokens provide granular control over specific blobs, queues, tables, or files.

#### Triage
Question: What is a SAS token in Azure Storage?
Probing: The interviewer might be assessing your understanding of Azure storage security and access mechanisms.

#### Assumptions
You should understand basic concepts like Azure storage accounts, Blob Storage, Queue Storage, Table Storage, and authentication methods.

#### Solution

Understanding SAS Tokens:

A SAS token is a URL-based security mechanism that enables you to grant limited access to specific resources within an Azure Storage account.
It works by adding a query string to the URL of your storage resource, which contains a set of permissions and a time range during which the token is valid.

Creating a SAS Token:

You can create a SAS token using tools like the Azure Portal, Azure CLI, or programming languages such as Python.
The essential components include the resource URI (full URL to your storage resource), a shared access policy with defined permissions and expiry time, and the storage account key.

Key Differences from Storage Account Key:

Granularity: SAS tokens allow you to specify exact permissions at the resource level, whereas storage account keys grant full access to the entire storage account.
Security: Using SAS tokens reduces the risk of exposing your full storage account key since each token has limited scope and duration.
Flexibility: You can dynamically generate and revoke SAS tokens as needed without changing any configuration in the Azure portal.

#### Example
Usage:

PYTHON
import os
from azure.storage.blob import BlobServiceClient, generate_blob_sas

connection_string = "your_connection_string"
container_name = "your_container_name"
blob_name = "your_blob_name"

# Create a client to interact with the storage account
blob_service_client = BlobServiceClient.from_connection_string(connection_string)

# Generate SAS token for a specific blob
sas_token = generate_blob_sas(
    account_name=blob_service_client.account_name,
    container_name=container_name,
    blob_name=blob_name,
    account_key=blob_service_client.credential.key,
    permission="r",  # Read-only access
    expiry=datetime.datetime.utcnow() + datetime.timedelta(hours=1)  # Token expires in 1 hour
)

# Construct the full URL with SAS token
sas_url = f"https://{blob_service_client.account_name}.blob.core.windows.net/{container_name}/{blob_name}?{sas_token}"

print("Generated SAS URL:", sas_url)

Best Practices:

Always use SAS tokens over storage account keys for granting access to specific resources.
Regularly update and revoke SAS tokens to maintain security.

#### Closing Summary

A Shared Access Signature (SAS) token in Azure Storage is a secure way to provide limited, time-bound access to your storage resources without exposing your full storage account key. It offers granular control over permissions and expiry, making it more flexible and safer than using storage account keys for all operations.

#### Common Mistakes/Follow-Ups
Misusing SAS Tokens: Using them incorrectly or not understanding the scope of granted permissions.
Expiring Tokens: Not regularly updating or revoking expired tokens.
Security Risks: Failing to secure the generation and handling process of SAS tokens, leading to potential leaks.

**Code:**

```python
import os
from azure.storage.blob import BlobServiceClient, generate_blob_sas

connection_string = "your_connection_string"
container_name = "your_container_name"
blob_name = "your_blob_name"

# Create a client to interact with the storage account
blob_service_client = BlobServiceClient.from_connection_string(connection_string)

# Generate SAS token for a specific blob
sas_token = generate_blob_sas(
    account_name=blob_service_client.account_name,
    container_name=container_name,
    blob_name=blob_name,
    account_key=blob_service_client.credential.key,
    permission="r",  # Read-only access
    expiry=datetime.datetime.utcnow() + datetime.timedelta(hours=1)  # Token expires in 1 hour
)

# Construct the full URL with SAS token
sas_url = f"https://{blob_service_client.account_name}.blob.core.windows.net/{container_name}/{blob_name}?{sas_token}"

print("Generated SAS URL:", sas_url)
```

