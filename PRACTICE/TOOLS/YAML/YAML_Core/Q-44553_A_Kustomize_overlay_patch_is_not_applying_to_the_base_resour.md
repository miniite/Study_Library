---
type: "practice"
difficulty: "Deep"
---

# A Kustomize overlay patch is not applying to the base resource. Explain how Kustomize matches patches by name/kind/namespace and how a YAML mismatch in those identity fields silently no-ops the patch.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kustomize Patching](<../../../../TOOLS/YAML/YAML_Core/Kustomize_Patching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In this scenario, we need to understand how Kustomize applies patches using the name, kind, and namespace fields in YAML resources. A mismatch between these identity fields can lead to a patch being silently ignored without any error messages. This understanding is crucial for debugging and ensuring that our configurations are applied correctly.

#### Triage
The question asks us to explain Kustomize's matching mechanism for patches and why a silent no-op occurs when there’s a mismatch in the name, kind, or namespace fields of YAML resources. We will assume that the base resource and patch share the same namespace, but their names or kinds might differ.

#### First Principles Solution

Understanding Kustomize Matching: Kustomize matches patches based on three primary fields: name, kind, and namespace. If all these match, it considers applying the patch to the base resource.
Mismatches Result in Silent No-ops: When there’s a mismatch in any of these identity fields (e.g., different names or kinds), Kustomize does not apply the patch at all. This can lead to confusion if the developer is expecting changes to be applied.

Let's break down an example:

#### Example

Base Resource:

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
namespace: default
spec:
  replicas: 3

Patch (Kustomization):

YAML
- op: replace
  path: /spec/replicas
  value: 5

In this case, Kustomize would apply the patch because name, kind, and namespace match.

Now consider a scenario where the namespace or name differs:

Scenario with Namespace Mismatch:

Base Resource:

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
namespace: production
spec:
  replicas: 3

Patch (Kustomization):

YAML
- op: replace
  path: /spec/replicas
  value: 5

Here, Kustomize will not apply the patch because of the mismatch in namespace.

Scenario with Kind Mismatch:

Base Resource:

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
namespace: default
spec:
  replicas: 3

Patch (Kustomization):

YAML
- op: replace
  path: /spec/replicas
  value: 5

In this case, Kustomize will also not apply the patch because of the mismatch in kind (assuming a different kind like Pod was used).

Scenario with Name Mismatch:

Base Resource:

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-new
namespace: default
spec:
  replicas: 3

Patch (Kustomization):

YAML
- op: replace
  path: /spec/replicas
  value: 5

Again, Kustomize will not apply the patch due to the mismatch in name.

Summary of Common Mistakes and Follow-Ups:
Assuming Mismatched Fields: Developers often overlook that a simple name or kind mismatch can cause patches to fail silently.
Namespace Mismatch Ignored: Be cautious about namespace mismatches, as Kustomize does not require the patch's namespace field to match the base resource’s namespace for it to be ignored.
Debugging Tips: Use Kustomize’s logging and verbose mode (--log-level=debug) to track which patches are being applied or not.

By understanding these principles, we can avoid common pitfalls and ensure our configurations are correctly applied in Kustomize overlays.

**Code:**

```yaml
# Example Base Resource
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
namespace: default
spec:
  replicas: 3

# Example Patch (Kustomization)
- op: replace
  path: /spec/replicas
  value: 5
```

**Example input / output:**

```text
Input:
Base Resource:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
namespace: default
spec:
  replicas: 3

Patch (Kustomization):
- op: replace
  path: /spec/replicas
  value: 5

Output:
Deployment "my-app" updated with spec.replicas set to 5.
```

