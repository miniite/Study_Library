---
type: "practice"
difficulty: "Deep"
---

# How do you inject secrets into a serverless function (Lambda/Cloud Run) that has cold starts, without Vault Agent sidecars?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bootstrapping Vault](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Bootstrapping_Vault.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To inject secrets into a serverless function like AWS Lambda or Google Cloud Run, especially in scenarios where these functions experience "cold starts," we need to ensure that the necessary secrets are accessible at runtime without relying on sidecars like Vault Agents. This involves leveraging managed services and secure storage mechanisms provided by cloud providers.

#### Triage

The question is about securely injecting secrets into serverless functions during cold starts, specifically avoiding the use of Vault Agents as sidecars. The core challenge here revolves around ensuring that the required secrets are available in memory or environment variables at function initialization, without introducing unnecessary complexity or performance overhead.

#### First-Principles Solution
Use Managed Secrets Stores: Leverage managed services provided by cloud providers such as AWS Secrets Manager for Lambda or Google Cloud Secret Manager for Cloud Run.
Environment Variables and IAM Roles: Utilize environment variables to pass secrets securely, often through IAM roles that grant the necessary permissions to access these secrets.
Invoke-Time Configuration: Ensure secrets are fetched from the managed secret store during function invocation.
Step-by-Step Solution

Store Secrets Securely:

Store your secrets in a cloud-managed service like AWS Secrets Manager or Google Cloud Secret Manager. These services provide strong encryption and secure access controls.

Configure IAM Roles:

Grant the Lambda/Cloud Run function an IAM role that has permission to read from the secret store.

Environment Variable Injection:

When deploying your serverless function, configure it to automatically fetch secrets from the managed secret store and inject them as environment variables.

Lambda Example with AWS Secrets Manager:

PYTHON
import boto3

def lambda_handler(event, context):
    # Initialize the Secrets Manager client
    secrets_client = boto3.client('secretsmanager')

    # Fetch a specific secret
    secret_name = 'your-secret-name'
    response = secrets_client.get_secret_value(SecretId=secret_name)

    # Parse and use the secret value as needed
    if 'SecretString' in response:
        secret_value = response['SecretString']
    else:
        secret_value = response['SecretBinary'].decode('utf-8')

    return {
        'statusCode': 200,
        'body': secret_value
    }

Cloud Run Example with Google Cloud Secret Manager:

Similar to AWS, use the google-cloud-secret-manager library.
PYTHON
from google.cloud import secretmanager

def cloud_run_function(request):
    # Initialize the Secret Manager client
    client = secretmanager.SecretManagerServiceClient()

    # Fetch a specific secret
    name = 'projects/your-project/secrets/your-secret-name/versions/latest'
    response = client.access_secret_version(name=name)

    # Parse and use the secret value as needed
    secret_value = response.payload.data.decode('UTF-8')

    return {
        "body": secret_value
    }

Handling Cold Starts:

Ensure that secrets are fetched during each invocation, not just at deployment or initialization. This ensures that if the function goes cold and is re-executed, it still has access to the necessary secrets.

#### Closing Summary

By leveraging managed services like AWS Secrets Manager or Google Cloud Secret Manager, we can securely store and inject secrets into serverless functions without relying on sidecars like Vault Agents. The key steps involve configuring IAM roles for secure access, fetching secrets during each function invocation, and ensuring that environment variables are correctly set.

#### Common Mistakes/Follow-Ups
Storing Secrets in Code: Avoid hardcoding secrets directly in the code as it reduces security.
Permissions Management: Ensure that the IAM roles have the minimum necessary permissions to avoid over-provisioning.

#### Performance Considerations
While managed services provide secure access, they can introduce latency during the first invocation. Monitor performance and optimize as needed.

**Comparison:**

```
Service	Action
AWS Secrets Manager	get_secret_value
Google Cloud Secret Manager	access_secret_version
```

**Code:**

```python
# AWS Lambda Example
import boto3

def lambda_handler(event, context):
    secrets_client = boto3.client('secretsmanager')

    secret_name = 'your-secret-name'
    response = secrets_client.get_secret_value(SecretId=secret_name)

    if 'SecretString' in response:
        secret_value = response['SecretString']
    else:
        secret_value = response['SecretBinary'].decode('utf-8')

    return {
        'statusCode': 200,
        'body': secret_value
    }

# Cloud Run Example
from google.cloud import secretmanager

def cloud_run_function(request):
    client = secretmanager.SecretManagerServiceClient()
    name = 'projects/your-project/secrets/your-secret-name/versions/latest'
    response = client.access_secret_version(name=name)

    secret_value = response.payload.data.decode('UTF-8')

    return {
        "body": secret_value
    }
```

**Example input / output:**

```text
# Lambda Example Input/Output
Input:
{
    "event": {},
    "context": {}
}

Output:
{
    'statusCode': 200,
    'body': 'SecretValue123'
}

# Cloud Run Example Input/Output
Request:
GET /example-endpoint

Response:
{
    "body": "SecretValue123"
}
```

**Data:**

```json
{
  "project_id": "example-project",
  "secret_name": "example-secret-name"
}
```

**Flow:**

1. Invoke Lambda/Cloud Run
2. Fetch Secret from ManagedStore
3. Set Environment Variables
4. Execute Function with Secrets

