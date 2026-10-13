---
type: "practice"
difficulty: "Deep"
---

# How would you design a shadow-deployment on the cloud where a new model version scores live traffic in parallel without affecting user responses?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tenant Isolation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Tenant_Isolation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a shadow-deployment system on the cloud that allows scoring of a new model version against live traffic without impacting user responses, we need to consider several key aspects: ensuring minimal downtime or degradation in service, managing data flow efficiently, and maintaining redundancy and fault tolerance. This requires careful architecture planning and implementation.

The core idea is to set up two parallel systems—live and shadow—that operate independently but share the same input data stream. The live system serves user requests directly, while the shadow system processes the same traffic in the background to generate new model predictions. Once the shadow system validates its performance, it can be seamlessly switched with the live system.

**Comparison:**

```
Component	Description
Live Model Deployment	Serves current version of model directly to users.
Shadow Model Deployment	Processes and scores new model version on a separate instance.
Ingress Controller / API Gateway	Routes traffic between live and shadow deployments as needed.
Monitoring & Validation System	Ensures shadow model performs adequately before switching.
```

**Code:**

```python
# Example of a basic setup using Kubernetes for container orchestration
import kubernetes.client as kube_client
from kubernetes import client, config

# Load configuration and create API clients
config.load_kube_config()
apps_v1 = client.AppsV1Api()

def deploy_shadow_deployment(current_model_name: str, new_model_name: str):
    # Define the deployment object for the shadow model
    deployment_obj = {
        "apiVersion": "apps/v1",
        "kind": "Deployment",
        "metadata": {
            "name": f"{new_model_name}-shadow"
        },
        "spec": {
            "replicas": 1,
            "selector": {
                "matchLabels": {"app": new_model_name, "version": "shadow"}
            },
            "template": {
                "metadata": {
                    "labels": {"app": new_model_name, "version": "shadow"}
                },
                "spec": {
                    "containers": [
                        {
                            "name": new_model_name,
                            "image": f"model_image:{new_model_name}",
                            "args": [f"--current_model={current_model_name}"]
                        }
                    ]
                }
            }
        }
    }

    # Deploy the shadow model
    apps_v1.create_namespaced_deployment(namespace="default", body=deployment_obj)
```

**Example input / output:**

```text
Request to the live system:
- Input Data: User request with features [1.2, 0.5, ...]
- Response: Model prediction based on current version

Shadow system processing:
- Same input data [1.2, 0.5, ...] passed through new model version
- Output: New predictions generated and stored for comparison

Switching process:
- Once validation is complete:
  - Set shadow deployment to be the live one
  - Update routing rules in the ingress controller or API gateway
```

**Data:**

```yaml
# Example deployment configuration for Kubernetes
apiVersion: apps/v1
kind: Deployment
metadata:
  name: new-model-shadow
spec:
  replicas: 1
  selector:
    matchLabels:
      app: new-model
      version: shadow
  template:
    metadata:
      labels:
        app: new-model
        version: shadow
    spec:
      containers:
      - name: new-model
        image: model_image:new-model
        args:
        - --current_model=current-model
```

**Flow:**

1. Live to Shadow
2. Serve User Requests
3. Process New Data
4. Route Traffic
5. Switch to Shadow
6. Validate Performance
7. Approve
8. Live Model Deployment
9. Shadow Model Deployment
10. Ingress Controller/API Gateway
11. Monitoring & ValidationSystem

