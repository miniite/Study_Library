---
type: "practice"
difficulty: "Deep"
---

# How do you handle YAML config that must include both secrets and non-secrets, keeping the secrets encrypted in git while the file stays valid and applyable?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Linting YAML](<../../../../TOOLS/YAML/YAML_Core/Linting_YAML.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling YAML configurations that need to include both sensitive (secrets) and non-sensitive data, a common challenge is ensuring that the sensitive information remains secure while still being accessible during application deployment. The key idea here is to separate secrets from non-secrets using best practices such as Kubernetes Secrets or environment variables, keeping the encrypted secrets out of version control.

The question probes understanding of how to manage sensitive and non-sensitive data in a YAML configuration file, with an emphasis on security, usability, and maintainability. We need to make sure that:

Secrets are stored securely.
The application can still read these secrets during deployment.
Non-secret data remains intact and easily manageable.

To achieve this, we will use Kubernetes Secrets for storing the encrypted secrets and reference them within our YAML configuration files using environment variables or other methods provided by the orchestration tooling.

#### First-Principles Solution
Store Secrets Securely: Use a secure method to store your secrets outside of version control. For example, you can use kubectl create secret commands to generate a Kubernetes Secret object that contains your sensitive data.
Reference Secrets in YAML: In the application's deployment YAML file, we'll reference these secrets using environment variables or other placeholder mechanisms.
Environment Variables for Secrets: Use environment variables to inject secrets into your application configuration at runtime. This ensures that secrets are not stored within the source code repository.
Step-by-Step Solution

Generate Kubernetes Secret:

BASH
kubectl create secret generic my-secrets --from-literal=secret1=value1 --from-literal=secret2=value2

Update YAML Configuration to reference secrets as environment variables or use a placeholder mechanism.

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  template:
    spec:
      containers:
        - name: my-container
          image: my-image:latest
          env:
            - name: SECRET1
              valueFrom:
                secretKeyRef:
                  name: my-secrets
                  key: secret1
            - name: SECRET2
              valueFrom:
                secretKeyRef:
                  name: my-secrets
                  key: secret2

Apply Configuration using kubectl apply to ensure the changes take effect.

BASH
kubectl apply -f deployment.yaml

#### Closing Summary
and Common Mistakes

In summary, managing sensitive data in YAML configurations requires a multi-step approach involving secure storage, proper referencing within the configuration files, and leveraging Kubernetes features like Secrets. Common mistakes include:

Storing secrets directly within the YAML file.
Failing to reference secrets properly at deployment time.
Not using environment variables or similar mechanisms to handle dynamic secret values.

By following these best practices, you can maintain a balance between security and operational efficiency in your application deployments.

**Comparison:**

```
Step	Action
1	Generate Kubernetes Secret object.
2	Update YAML file to reference secret as environment variable.
3	Apply updated configuration.
```

**Code:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  template:
    spec:
      containers:
        - name: my-container
          image: my-image:latest
          env:
            - name: SECRET1
              valueFrom:
                secretKeyRef:
                  name: my-secrets
                  key: secret1
            - name: SECRET2
              valueFrom:
                secretKeyRef:
                  name: my-secrets
                  key: secret2
```

**Example input / output:**

```text
# Generate Kubernetes Secret
kubectl create secret generic my-secrets --from-literal=secret1=value1 --from-literal=secret2=value2

# Apply the updated deployment configuration
kubectl apply -f deployment.yaml
```

**Data:**

```json
{
  "secrets": {
    "secret1": "value1",
    "secret2": "value2"
  }
}
```

**Flow:**

1. Generate Secrets
2. Update YAML for Secrets
3. Apply Configuration

