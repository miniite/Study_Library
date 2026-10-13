---
type: "practice"
difficulty: "Deep"
---

# You want CI to fail a PR that adds a Kubernetes manifest without resource requests/limits, before it ever reaches Git main. Would you use a conftest/OPA policy in CI, an admission controller, or both, and where does each catch what the other misses?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Isolating CI Environments](<../../../../TOOLS/Git_and_GitHub/Git_Core/Isolating_CI_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that PRs adding Kubernetes manifests without resource requests/limits fail CI before reaching Git main, we need to consider using conftest and OPA (Open Policy Agent) policies in CI, as well as admission controllers in the Kubernetes cluster itself. Each of these tools serves a specific purpose and has its own strengths and weaknesses.

TL;DR: Use both conftest/OPA policies in CI and an admission controller in the Kubernetes cluster to catch all cases where resource requests/limits are missing. Conftest is better at static analysis and can run quickly, while OPA can provide more dynamic enforcement based on complex policies.

**Comparison:**

```
Tool	Purpose	When to Use
Conftest	Static analysis, run CI tests	Early in the CI pipeline, before merging PRs
Admission Controller	Dynamic enforcement, real-time validation	In Kubernetes cluster to block creation of non-compliant resources
```

**Code:**

```yaml
# Example conftest policy for checking resource requests/limits
apiVersion: conftest.k8s.io/v1beta1
kind: Check
metadata:
  name: missing-resource-check
spec:
  path: .
  include:
    - '**/*.yaml'
  checks:
    - matchers:
        - operator: hasRequestOrLimit
          selector: .spec.template.spec.containers[*]
```

**Example input / output:**

```
Input (manifest with no resource requests/limits):
YAML
Copy
---
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - image: nginx:latest
Output from conftest:
PLAINTEXT
Copy
Error: /path/to/manifest.yaml:3:27: container 'nginx' does not have resource requests or limits defined.
```

**Data:**

```json
{
  "manifest": {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "example-pod"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:latest",
          "resources": {}
        }
      ]
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Conftest runs
6. Missing resource?
7. Fail CI
8. Pass CI
9. PR merged into main
10. Admission controller runs
11. Missing resource?
12. Reject PR in cluster
13. Resource created/updated

