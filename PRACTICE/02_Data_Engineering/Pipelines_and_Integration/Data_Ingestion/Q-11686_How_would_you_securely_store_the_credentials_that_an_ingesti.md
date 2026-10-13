---
type: "practice"
difficulty: "Intermediate"
---

# How would you securely store the credentials that an ingestion tool uses to connect to a source?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Ingestion Credentials](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Ingestion_Credentials.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To securely store the credentials used by an ingestion tool like Airbyte or Fivetran, we need to ensure that sensitive information such as API keys and connection details are not exposed in plaintext within our codebase or configuration files. The goal is to minimize security risks while still allowing for seamless data integration.

The question here revolves around securing secrets in a way that balances security with operational efficiency. Key assumptions include the use of an ingestion tool like Airbyte, which typically requires authentication and authorization details. We need to consider where these credentials are stored, how they are accessed by the ingestion process, and what best practices can be employed to mitigate risks.

Environment Variables: Store sensitive data in environment variables. This approach keeps secrets out of your source code or configuration files.
Secret Management Tools: Use secret management tools like HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, or Google Cloud Secret Manager.
Encryption at Rest and In Transit: Ensure that any stored credentials are encrypted both at rest and in transit to prevent unauthorized access.

Let's go through these steps:

Step 1: Environment Variables

Using environment variables is a simple and effective way to manage secrets. You can set these variables in your CI/CD pipeline, local development environments, or infrastructure as code (IaC) files like Terraform or Ansible playbooks.

#### Example
SH
export AIRBYTE_SOURCE_API_KEY="your_api_key_here"
Step 2: Secret Management Tools

Secret management tools provide a centralized repository to store and manage secrets. They offer features such as versioning, lifecycle management, and automatic rotation of secrets. This ensures that even if an attacker gains access to one instance of a secret, the risk is mitigated.

#### Example
(HashiCorp Vault):
SH
# Initialize and unseal Vault
vault operator init -key-shares=1 -key-threshold=1
vault unseal <unseal_key>

# Store secrets in Vault
vault write secret/airbyte/api-key api_key="your_api_key_here"
Step 3: Encryption at Rest and In Transit

Ensure that the stored secrets are encrypted both when they are stored and when they are transmitted. This can be achieved using tools like AWS KMS, Azure Key Vault, or similar services.

#### Example
(AWS Secrets Manager):
PYTHON
import boto3

client = boto3.client('secretsmanager')
response = client.get_secret_value(SecretId='airbyte-api-key')
api_key = response['SecretString']['api_key']

#### Implementation
in Airbyte/Fivetran

For an ingestion tool like Airbyte, you would configure the connection settings to use these stored secrets. For example, in Airbyte's UI or configuration file, you would reference the environment variables:

YAML
source:
  type: custom_source
  config:
    api_key: $AIRBYTE_SOURCE_API_KEY

destination:
  type: custom_destination
  config:
    api_key: $AIRBYTE_DESTINATION_API_KEY

#### Closing Summary
+ Common Mistakes/Follow-Ups

Storing credentials securely is crucial to maintaining data integrity and compliance. Common mistakes include hardcoding secrets into scripts or configuration files, using insecure methods like plain text files for storing secrets, or neglecting encryption at rest and in transit.

To address these issues, always use environment variables, secret management tools, and ensure encryption practices are followed diligently. Follow-up questions might probe deeper into specific technologies used, additional security measures (e.g., multi-factor authentication), and how to handle secret rotations and access controls.

**Code:**

```python
import os

api_key = os.getenv('AIRBYTE_SOURCE_API_KEY')
```

