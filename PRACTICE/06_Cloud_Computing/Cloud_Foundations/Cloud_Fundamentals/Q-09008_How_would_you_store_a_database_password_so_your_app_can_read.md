---
type: "practice"
difficulty: "Intermediate"
---

# How would you store a database password so your app can read it but it never appears in your code repo?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Secrets and Key Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To securely store and manage database passwords for an application, ensuring they do not appear directly in the code repository, we need to employ a combination of environment variables and secure vaults or secrets management tools. This approach ensures that sensitive data like passwords are kept out of source control, making them harder to accidentally expose.

The question is probing your understanding of security best practices in DevOps environments, specifically around managing sensitive information such as database credentials. Key assumptions include the use of a modern cloud environment (e.g., AWS, GCP) and access management for your application.

#### First-Principles Solution
Environment Variables: Store the password as an environment variable.
Secrets Management Tools: Use tools like AWS Secrets Manager, Azure Key Vault, or HashiCorp Vault to securely store and retrieve the secrets.
Dynamic Configuration: Load the configuration dynamically at runtime from these tools rather than hardcoding it in your application.
Step-by-Step Explanation
1. Environment Variables

The first step is to avoid storing any sensitive data directly within the codebase. Instead, set up environment variables for the database password and other secrets. You can define them in a configuration file or command-line arguments that are sourced when starting your application.

For example:

BASH
DB_PASSWORD=my_secure_password
2. Secrets Management Tools

Secrets management tools provide an encrypted storage mechanism where sensitive data is stored. These tools support features like encryption, access control, and versioning of secrets.

AWS Secrets Manager
Create a Secret: Use the AWS Console or CLI to create a new secret.
SH
aws secretsmanager create-secret --name my-app-db-secret --secret-string '{"username":"admin","password":"my_secure_password"}'
Retrieve the Secret in your application:
PYTHON
import boto3

client = boto3.client('secretsmanager')
response = client.get_secret_value(SecretId='my-app-db-secret')
db_password = response['SecretString']['password']
Azure Key Vault
Create a Secret: In the Azure Portal, create a new secret.
Retrieve the Secret in your application:
PYTHON
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

credential = DefaultAzureCredential()
client = SecretClient(vault_url="https://mykeyvault.vault.azure.net/", credential=credential)
db_password = client.get_secret('my-app-db-secret').value
HashiCorp Vault
Create a Secret: Use the Vault CLI to create a new secret.
SH
vault write secret/my-app/db-password password=my_secure_password
Retrieve the Secret in your application:
BASH
export DB_PASSWORD=$(vault read -field=password secret/my-app/db-password)
3. Dynamic Configuration

Load the configuration dynamically at runtime using environment variables or by reading from a secrets management tool.

#### Example
Python Code to Connect to Database Using Secrets Manager
PYTHON
import os

db_username = os.getenv('DB_USERNAME')
db_password = os.environ['DB_PASSWORD']

# Use db_username and db_password in your application

#### Closing Summary
+ Common Mistakes/Follow-Ups

Storing database passwords outside the code repository is crucial for security. Using environment variables and secrets management tools ensures that sensitive information remains secure and accessible only to authorized users. Some common mistakes include:

Directly hardcoding passwords in the code.
Storing secrets in plain text files or version control systems.
Not rotating keys and secrets regularly.

#### Follow-ups
could involve discussing how to handle secret rotations, multi-environment support (e.g., dev, staging, prod), and integrating with CI/CD pipelines for secure deployment practices.

**Comparison:**

```
Secrets Management Tool	Key Features
AWS Secrets Manager	Encrypted storage, access control, versioning, integration with IAM policies
Azure Key Vault	Secure vault management, RBAC, certificate management
HashiCorp Vault	Flexible secret storage, multi-factor authentication, integrations with CI/CD tools
```

**Code:**

```python
# Example Python Code to Connect to Database Using Secrets Manager
import boto3

client = boto3.client('secretsmanager')
response = client.get_secret_value(SecretId='my-app-db-secret')
db_password = response['SecretString']['password']

# Use db_username and db_password in your application
```

**Data:**

```json
{
  "username": "admin",
  "password": "my_secure_password"
}
```

**Flow:**

1. Yes
2. No
3. Define Environment Variables
4. Store Secrets in AWS SecretsManager
5. Retrieve Secret at Runtime?
6. Use Retrieved Secret toConnect to DB
7. Directly Hardcode Password(Bad Practice)

