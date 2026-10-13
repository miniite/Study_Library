---
type: "practice"
difficulty: "Deep"
---

# You run deploys via ArgoCD but a sync keeps flapping between healthy and progressing on one service; walk me through how you'd debug whether it's a bad health check, a mutating webhook, or resource-hash instability.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Push vs Pull Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When encountering an issue where a deployment via ArgoCD keeps oscillating between "healthy" and "progressing" states on a single service, the primary goal is to isolate whether the problem lies in the health check, a mutating webhook, or resource-hash instability. I would systematically approach this by debugging at different levels, from the application's perspective down to the infrastructure.

Firstly, let’s clarify what ArgoCD does: it ensures that your application’s environment matches the desired state defined in your GitOps configuration files. The health check determines whether a sync is considered healthy; meanwhile, resource hashes ensure consistency between expected and actual states. Mutating webhooks can modify resources as they are applied.

**Comparison:**

```
Service	Status	Details
service-a	progressing	last_healthy_sync: 10, current_reconcile: 20

service-b	healthy	-
```

**Code:**

```python
# Example Python function to simulate a simple health check result retrieval
def get_health_check_result(service_name):
    # Mock data for demonstration
    if service_name == "service-a":
        return {"status": "progressing", "details": {"last_healthy_sync": 10, "current_reconcile": 20}}
    else:
        return {"status": "healthy", "details": {}}

# Example function to simulate a resource hash check
def get_resource_hash(service_name):
    # Mock data for demonstration
    if service_name == "service-a":
        return "hash123"
    else:
        return None
```

**Example input / output:**

```

Given the YAML above, let’s simulate a health check result for service-a:

PYTHON
Copy
health_check_result = get_health_check_result("service-a")
print(health_check_result)
# Output:
# {"status": "progressing", "details": {"last_healthy_sync": 10, "current_reconcile": 20}}

Resource hash for service-a:

PYTHON
Copy
resource_hash = get_resource_hash("service-a")
print(resource_hash)
# Output:
# "hash123"
```

**Data:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
  namespace: default
spec:
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  project: default
  source:
    repoURL: git@github.com:example/example-repo.git
    path: manifests
    targetRevision: HEAD
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. No
6. No
7. Start Debugging
8. Is the issue related to a specificservice?
9. Check Health Check Results
10. Does the health check reflectapplication readiness?
11. Investigate Health CheckLogic/Configuration
12. Check Resource Hashes
13. Are resource hashesinconsistent between expectedand actual states?
14. Debug Resource HashCalculation
15. Verify Mutating Webhooks
16. Investigate InfrastructureConsistency

