---
type: "practice"
difficulty: "Basic"
---

# How would you store a database password that the pipeline needs, safely?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Secrets in Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To securely store and manage a database password in a CI/CD pipeline, we need to ensure that sensitive information is not exposed or stored insecurely. The key idea here is to use environment variables or encrypted secrets management tools that can be dynamically injected into the build process.

This question asks about storing a database password safely in a CI/CD context. The interviewer might be probing your understanding of security practices, best practices for managing sensitive data, and how you would integrate this with an existing CI/CD pipeline. We'll assume we are working in a Linux environment but these principles can apply to other operating systems as well.

Step-by-Step Solution:
Identify the Storage Mechanism: Use an encrypted secrets management tool or store the password as an environment variable.
Generate and Securely Store the Password:
Generate a strong, random password using tools like openssl or a dedicated secrets manager.
Store this password securely in your CI/CD system.
Use Environment Variables in Pipeline: Inject the password into your pipeline as an environment variable to avoid hardcoding it in scripts or configuration files.
Detailed Steps:

Generate and Encrypt the Password:

BASH
# Generate a strong random 32 character base64 encoded string
openssl rand -base64 32 > db_password.txt

# Store the password securely, for example using a secrets manager like HashiCorp Vault or AWS Secrets Manager.

Store in CI/CD System:

If you are using an open-source solution like Jenkins, GitLab, or GitHub Actions, store the password as an environment variable.
For enterprise solutions, use tools like HashiCorp Vault to manage secrets securely.

Inject into Pipeline Script:

BASH
# Example for a simple shell script in your pipeline
export DB_PASSWORD=$(cat /path/to/db_password.txt)

Run the Pipeline:

Ensure that the environment variable DB_PASSWORD is available during runtime.
The database connection string should look something like this: mysql://username:$DB_PASSWORD@localhost/database.

#### Closing Summary
and Common Mistakes:

Storing a database password safely in a CI/CD pipeline requires careful consideration of security practices. Common mistakes include hardcoding passwords directly into scripts or configuration files, which can lead to exposure during code reviews or version control history. Always use environment variables or encrypted secrets management tools to keep sensitive information secure.

Common Follow-Ups:
How do you handle situations where multiple services need access to the same database password?
What if the password needs to be changed? How would you manage that securely in a pipeline?

**Code:**

```bash
# Example of generating and storing a password
openssl rand -base64 32 > db_password.txt

# Example Jenkinsfile snippet for injecting an environment variable
pipeline {
    agent any
    environment {
        DB_PASSWORD = credentials('db-password-alias')
    }
    stages {
        stage('Build') {
            steps {
                echo "Using DB password: ${env.DB_PASSWORD}"
            }
        }
    }
}
```

**Data:**

```json
{
  "secrets_manager": {
    "type": "Vault",
    "path": "/secrets/db_password"
  },
  "environment_variable": "DB_PASSWORD=$(cat /path/to/secrets.json)"
}
```

