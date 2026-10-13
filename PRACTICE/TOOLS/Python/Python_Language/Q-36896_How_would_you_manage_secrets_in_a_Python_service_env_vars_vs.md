---
type: "practice"
difficulty: "Deep"
---

# How would you manage secrets in a Python service (env vars vs a secrets manager vs mounted files), and how do you rotate a DB credential without a restart or downtime?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Credential Handling](<../../../../TOOLS/Python/Python_Language/Credential_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When managing secrets in a Python service, choosing between environment variables, a secrets manager, or mounting files depends on your specific use case, security requirements, and operational constraints. Typically, environment variables are used for simpler applications with fewer secrets, while more complex environments may benefit from a dedicated secrets manager to handle encryption at rest and secure access. Secrets can be rotated without downtime using various techniques, such as zero-downtime deployments or dynamic secret replacement.

To manage secrets effectively in Python, we need to consider the following steps:

Environment Variables: Simplest approach for small services.
Secrets Manager (e.g., AWS Secrets Manager): Provides encryption and secure access mechanisms.
Mounted Files: Useful when secrets are part of the application's file system.

The goal is to ensure that secrets can be updated or rotated without causing service disruptions, which involves careful planning around how secrets are accessed and replaced in the running process.

**Comparison:**

```
Approach	Pros	Cons
Environment Variables	Simple, easy to set up for small applications.	Security risks if mismanaged or exposed in code.
Secrets Manager	Enhanced security features (encryption at rest, IAM policies).	Cost and complexity of setup; additional dependency on the secrets manager service.
Mounted Files	Flexibility to include secrets as part of file system.	Potential security risks if files are not properly secured or accessed improperly.
```

**Code:**

```python
import os

# Example of using environment variables for a secret
db_password = os.getenv('DB_PASSWORD')
```

**Example input / output:**

```
Rotating a Secret Using AWS Secrets Manager:
PYTHON
Copy
import boto3

secrets_client = boto3.client('secretsmanager')

def rotate_secret(secret_id):
    # Get current version of the secret
    current_version = secrets_client.get_secret_value(SecretId=secret_id)

    # Create a new version with updated credentials (e.g., DB password)
    new_version = secrets_client.create_secret(
        Name=secret_id,
        Description="Updated database credential",
        SecretString='{"username": "admin", "password": "new_secure_password"}'
    )
```

**Data:**

```
AWS Secrets Manager JSON Configuration:
JSON
Copy
{
  "SecretId": "my-database-secret",
  "VersionId": "1234567890abcdef1234567890abcdef"
}
```

**Flow:**

1. Yes
2. No
3. Initial Secret Load
4. Is it a small application?
5. Use Environment Variables
6. Consider Secrets Manager orMounted Files
7. Secret Rotation Process
8. Load New Secret ValueWithout Restart

