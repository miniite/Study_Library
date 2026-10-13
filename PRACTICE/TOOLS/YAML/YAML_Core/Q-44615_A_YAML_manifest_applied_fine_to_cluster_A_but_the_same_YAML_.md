---
type: "practice"
difficulty: "Deep"
---

# A YAML manifest applied fine to cluster A but the same YAML failed on cluster B. What version-skew or admission differences (not YAML syntax) would you investigate first?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kustomize Patching](<../../../../TOOLS/YAML/YAML_Core/Kustomize_Patching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a YAML manifest works seamlessly in one Kubernetes cluster but fails in another, the issue is likely rooted in fundamental differences between the two clusters rather than any syntax errors in the YAML itself. The key areas to investigate are version skew and admission controller configurations, as these can significantly affect how resources are interpreted and deployed.

**Comparison:**

```
Cluster	Kubernetes Version	Admission Controllers
A	v1.23.0	PodSecurityPolicy, LimitRanger, ServiceAccount
B	v1.25.0	PodSecurityPolicy, LimitRanger, ResourceQuota, NamespaceLifecycle
```

**Code:**

```
Example YAML manifest

apiVersion: v1 kind: Pod metadata: name: nginx-pod spec: containers: - name: nginx-container image: nginx:latest

Cluster A and B might have different Kubernetes versions or admission controller policies.

ClusterA_version = "v1.23.0" ClusterB_version = "v1.25.0"

ClusterA_admission_controllers = [ "PodSecurityPolicy", "LimitRanger", "ServiceAccount" ]

ClusterB_admission_controllers = [ "PodSecurityPolicy", "LimitRanger", "ResourceQuota", "NamespaceLifecycle" ]
```

**Example input / output:**

```
Example manifest with issues due to version skew or admission controller differences
For Kubernetes 1.23.0

apiVersion: v1 kind: Pod metadata: name: nginx-pod spec: containers: - name: nginx-container image: nginx:latest

CODE
Copy

```yaml
# For Kubernetes 1.25.0
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  securityContext:
    fsGroup: 2000
  containers:
    - name: nginx-container
      image: nginx:latest
```

**Data:**

```text
- Cluster A: Kubernetes version 1.23.0
- Cluster B: Kubernetes version 1.25.0
- Admission controllers in Cluster A: PodSecurityPolicy, LimitRanger, ServiceAccount
- Admission controllers in Cluster B: PodSecurityPolicy, LimitRanger, ResourceQuota, NamespaceLifecycle
```

**Flow:**

1. Kubernetes Version
2. Admission Controllers
3. Yes
4. No
5. Cluster A
6. Cluster B
7. Are the admission controllersdifferent?
8. Investigate version skew oradmission controllerdifferences in YAML manifest
9. Check for syntax errors orother issues not related toYAML versioning/admissioncontrol

