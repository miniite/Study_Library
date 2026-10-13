---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through securing the Vault-to-storage and Vault-to-client communication with TLS in a k8s deployment.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bootstrapping Vault](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Bootstrapping_Vault.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Kubernetes (k8s) environment, ensuring secure communications between Vault and storage, as well as between Vault and clients, is crucial. This involves implementing Transport Layer Security (TLS) to encrypt all data in transit. We will cover the setup of mutual TLS for Vault, integrating it with k8s secrets management, and securing access from both internal (storage) and external (client) services.

#### Triage

The question asks us to secure communications between Vault and storage, as well as between Vault and clients using TLS in a Kubernetes deployment. We need to ensure that data is encrypted in transit and that mutual authentication is implemented to protect against man-in-the-middle attacks.

#### Assumptions
Kubernetes Cluster: Running on either on-premises or cloud infrastructure.
Vault Deployment: Already configured within the k8s cluster with a secret engine for storing sensitive information.
Storage and Client Services: Both services are deployed in the same k8s namespace.
Step-by-Step Solution
1. Generate TLS Certificates

First, we need to generate self-signed or externally provided TLS certificates for Vault and storage/client services.

BASH
# Example: Generating a self-signed certificate using OpenSSL
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout vault-key.pem -out vault.crt

# For clients, you might want to generate or obtain certificates from a CA.
2. Configure Vault for TLS

Configure Vault to use the generated TLS certificate and key.

HCL
# Vault Configuration File (vault-config.hcl)
api_addr = "https://<VAULT_IP>:8200"
tls_disable = false

# Path for certificates and keys
certificate_path = "/path/to/vault.crt"
private_key_path = "/path/to/vault-key.pem"

# Enable a secrets engine if not already enabled
# vault enable <secret-engine-type>

# Example: Enable the kv-v2 secret engine
vault mount -storage=kvs /kv-v2
3. Create Kubernetes Secrets for Vault TLS Certificates

Create k8s secrets to store the Vault TLS certificate and key.

YAML
apiVersion: v1
kind: Secret
metadata:
  name: vault-tls-certs
type: Opaque
data:
  cert.crt: <base64-encoded-vault-certificate>
  key.key: <base64-encoded-vault-private-key>
4. Configure Vault to Use Kubernetes Secrets for TLS

Use the k8s secrets in Vault’s configuration.

HCL
# vault-config.hcl
api_addr = "https://<VAULT_IP>:8200"
tls_disable = false

# Mount path and secret name
kubernetes_secret_engine_path = "/secret/k8s/tls"
kubernetes_api_server_address = "<K8S_API_SERVER>"
kubernetes_namespace = "<NAMESPACE>"

# Use the secrets to configure Vault
certificate_path = "kubernetes_secret_engine_path/cert.crt"
private_key_path = "kubernetes_secret_engine_path/key.key"
5. Secure Communication Between Vault and Storage

To secure communication between Vault and storage, ensure that both services use mutual TLS.

YAML
# Example: Kubernetes Deployment for Storage Service (storage-deployment.yaml)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: storage-service
spec:
  selector:
    matchLabels:
      app: storage-service
  template:
    metadata:
      labels:
        app: storage-service
    spec:
      containers:
      - name: storage-service
        image: <storage-image>
        ports:
        - containerPort: 8080
        volumeMounts:
        - mountPath: /vault-certs
          name: vault-tls-certs
      volumes:
      - name: vault-tls-certs
        secret:
          secretName: vault-tls-certs

# Configure the service to use TLS and mutual authentication.
apiVersion: v1
kind: Service
metadata:
  name: storage-service
spec:
  selector:
    app: storage-service
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: ClusterIP
  tls:
    termination: edge
    secretName: vault-tls-secret
6. Secure Communication Between Vault and Client Services

Similarly, secure the communication between Vault and client services using mutual TLS.

YAML
# Example: Kubernetes Deployment for Client Service (client-deployment.yaml)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-service
spec:
  selector:
    matchLabels:
      app: client-service
  template:
    metadata:
      labels:
        app: client-service
    spec:
      containers:
      - name: client-service
        image: <client-image>
        ports:
        - containerPort: 8080
        volumeMounts:
        - mountPath: /vault-certs
          name: vault-tls-certs
      volumes:
      - name: vault-tls-certs
        secret:
          secretName: vault-tls-certs

# Configure the service to use TLS and mutual authentication.
apiVersion: v1
kind: Service
metadata:
  name: client-service
spec:
  selector:
    app: client-service
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: ClusterIP
  tls:
    termination: edge
    secretName: vault-tls-secret
7. Test the Setup

Perform a dry-run to ensure that all services can communicate with Vault using mutual TLS.

BASH
# Example: Client Service Testing
curl -k https://<VAULT_IP>/kv-v2 -H "Host: <client-service-hostname>" --cert /path/to/client.crt --key /path/to/client.key

# Example: Storage Service Testing
curl -k https://<VAULT_IP>/kv-v2 -H "Host: <storage-service-hostname>" --cert /path/to/storage.crt --key /path/to/storage.key

#### Closing Summary
and Common Mistakes/Follow-Ups

By configuring mutual TLS for both Vault to storage and Vault to client communications, we ensure secure data in transit. The key steps include generating certificates, configuring Vault, creating k8s secrets, and ensuring that all services use the correct certificates.

#### Common mistakes
include:

Not properly configuring the tls_disable flag.
Forgetting to mount TLS secrets correctly within container volumes.
Misconfiguring service definitions without proper tls termination.

For follow-ups, you might be asked about scaling considerations, managing certificate lifetimes, or integrating with a third-party CA for more robust certificate management.

**Comparison:**

```
Component	Configuration
Vault	TLS enabled, mutual authentication configured
Storage	Mutual TLS enabled for secure communication
Client	Mutual TLS enabled for secure communication
```

**Code:**

```hcl
# Vault Configuration File (vault-config.hcl)
api_addr = "https://<VAULT_IP>:8200"
tls_disable = false

certificate_path = "/path/to/vault.crt"
private_key_path = "/path/to/vault-key.pem"

kubernetes_secret_engine_path = "/secret/k8s/tls"
kubernetes_api_server_address = "<K8S_API_SERVER>"
kubernetes_namespace = "<NAMESPACE>"

# Path for certificates and keys
certificate_path = "kubernetes_secret_engine_path/cert.crt"
private_key_path = "kubernetes_secret_engine_path/key.key"
```

**Example input / output:**

```text
# Example: Client Service Testing
curl -k https://<VAULT_IP>/kv-v2 -H "Host: <client-service-hostname>" --cert /path/to/client.crt --key /path/to/client.key

# Example: Storage Service Testing
curl -k https://<VAULT_IP>/kv-v2 -H "Host: <storage-service-hostname>" --cert /path/to/storage.crt --key /path/to/storage.key
```

**Data:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: vault-tls-certs
type: Opaque
data:
  cert.crt: <base64-encoded-vault-certificate>
  key.key: <base64-encoded-vault-private-key>
```

**Flow:**

1. Generate Certificates
2. Configure Vault
3. Create k8s Secrets
4. Deploy Storage Service withTLS
5. Deploy Client Service with TLS
6. Test Setup
7. Verify Secure Communication

