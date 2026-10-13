---
type: "practice"
difficulty: "Basic"
---

# Why is it dangerous to bake a Vault token into a container image, and what pattern replaces it?

**Difficulty:** Basic · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bootstrapping Vault](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Bootstrapping_Vault.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When integrating a Vault token directly into a container image, you're essentially hardcoding sensitive information right in the Dockerfile or baked into the image itself. This approach is perilous because once the secret is exposed, it can be extracted and misused by unauthorized parties. A safer pattern involves using environment variables or leveraging Kubernetes secrets to manage Vault tokens securely.

**Code:**

```bash
# Example of hardcoding a Vault token in Dockerfile (DO NOT DO THIS)
FROM alpine
ARG VAULT_TOKEN=your-vault-token-here
RUN echo $VAULT_TOKEN > /path/to/token/file
CMD ["tail", "-f", "/dev/null"]
```

**Example input / output:**

```

To replace hardcoded tokens, you would instead fetch the token from a secure location. Here’s an example of using environment variables in a Dockerfile and a Kubernetes deployment:

Dockerfile

DOCKERFILE
Copy
FROM alpine
ARG VAULT_ADDR=https://vault.example.com
RUN apk add --no-cache curl && \
    echo "VAULT_TOKEN=$(cat /run/secrets/${VAULT_TOKEN_ENV_VAR})" > /etc/profile.d/vault-token.sh
CMD ["sh", "/etc/profile.d/vault-token.sh"]

Kubernetes Deployment

YAML
Copy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  template:
    spec:
      containers:
      - name: my-app-container
        image: my-app-image
        env:
          - name: VAULT_TOKEN_ENV_VAR
            valueFrom:
              secretKeyRef:
                name: vault-secrets
                key: token
```

**Data:**

```json
{
  "token": "your-vault-token-here",
  "envVarName": "VAULT_TOKEN"
}
```

