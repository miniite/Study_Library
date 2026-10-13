---
type: "practice"
difficulty: "Basic"
---

# Why do we prefer Vault's dynamic database secrets over storing a long-lived static DB password in a Kubernetes Secret?

**Difficulty:** Basic · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of Env and Logs](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Keeping_Secrets_out_of_Env_and_Logs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When it comes to managing sensitive information like database passwords in Kubernetes, using HashiCorp Vault’s dynamic database secrets is significantly more secure and flexible compared to storing a long-lived static password directly in a Kubernetes Secret. This approach not only mitigates the risks associated with static secrets but also provides better scalability and compliance.

Vault's dynamic secrets offer several advantages:

Rotation: Dynamic secrets can be automatically rotated, reducing the risk of exposure due to compromised credentials.
Security: Vault manages encryption keys securely, ensuring that sensitive data remains protected even if someone gains access to the Kubernetes Secret.
Auditability: With Vault, you have detailed logs and audit trails for all secret usage and rotations, which is crucial for compliance.

To understand why dynamic secrets are preferred, let’s break down the problem from first principles and walk through a step-by-step solution.

**Comparison:**

```
Step	Action
1	Initialize Vault and set up a secret engine.
2	Request a database connection from the application.
3	Validate credentials with Vault.
4	Generate or retrieve dynamic password.
5	Return detailed connection information to the application.
```

**Code:**

```bash
# Example of storing a static password in a Kubernetes Secret
apiVersion: v1
kind: Secret
metadata:
  name: db-password-secret
type: Opaque
data:
  username: dXNlcm5hbWU=
  password: cGFzc3dvcmQ= # This is an example static password

# Example of using Vault to generate dynamic secrets for a database connection
```

**Example input / output:**

```

When you need to connect to a database, instead of hardcoding the static password into your application or Kubernetes Secret, you can use Vault to dynamically generate and retrieve the password. Here’s an example input-output flow:

Initial Setup: Configure Vault and set up a secret engine for database secrets.
Request from Application:
Your application requests a new database connection string.
The application sends its authentication details to Vault.
Vault Response:
Vault checks the credentials and generates or retrieves a dynamic password if it exists.
Vault returns the generated or retrieved password along with other necessary information like the host, port, etc., in JSON format.
JSON
Copy
{
  "username": "myuser",
  "password": "randomly-generated-password",
  "host": "database.example.com",
  "port": 3306
}
```

**Data:**

```json
{
  "username": "myuser",
  "password": "randomly-generated-password"
}
```

**Flow:**

1. Yes
2. No
3. Application
4. Vault
5. Valid Credentials?
6. Generate Dynamic Password
7. Fail Authentication
8. Return Connection Details toApplication

