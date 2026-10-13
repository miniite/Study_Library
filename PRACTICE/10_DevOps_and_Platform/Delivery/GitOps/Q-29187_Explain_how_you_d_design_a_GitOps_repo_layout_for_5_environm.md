---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a GitOps repo layout for 5 environments and 3 regions without duplicating YAML across 15 combinations.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Repo and Environment Layout](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In designing a GitOps repository structure for five environments (e.g., development, staging, production) and three regions (e.g., us-east-1, eu-west-1, ap-southeast-1), the key challenge is to ensure that YAML configurations are not duplicated across 15 combinations. This can be achieved through a hierarchical directory structure that leverages environment-specific overlays, region-specific variables, and shared base configurations.

#### Triage
The question is probing your understanding of GitOps principles, particularly around how to manage infrastructure as code (IaC) in multiple environments while maintaining consistency and avoiding redundancy. Key assumptions include the use of a templating engine like Helm or Jinja2 for YAML files, and a version control system like Git.

#### First Principles Solution

Repository Structure: Design a hierarchical directory structure that separates base configurations from environment-specific overlays.
Base Configurations: Store shared resources in a central base directory. These are common across all environments and regions.
Environment-Specific Overlays: Create directories for each environment, which contain YAML overrides or additional resources specific to that environment.
Region-Specific Variables: Use Helm charts or Jinja2 templates to allow region-specific variable values without duplicating entire files.

Step-by-Step Solution:

Repository Root Structure:

CODE
gitops-repo/
├── base/
│   └── kubernetes/
│       └── common.yaml
├── environments/
│   ├── dev/
│   │   └── kubernetes/
│   │       └── overlay.yaml
│   ├── staging/
│   │   └── kubernetes/
│   │       └── overlay.yaml
│   └── prod/
│       └── kubernetes/
│           └── overlay.yaml
└── regions/
    ├── us-east-1/
    │   └── values.yaml
    ├── eu-west-1/
    │   └── values.yaml
    └── ap-southeast-1/
        └── values.yaml

Base Configurations:

common.yaml contains the base configuration shared across all environments and regions.
YAML
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: nginx:latest

Environment-Specific Overlays:

overlay.yaml in each environment directory contains overrides or additional resources specific to that environment.
YAML
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: nginx:latest

Region-Specific Variables:

values.yaml in the regions directory contains region-specific variable values.
YAML
region: us-east-1
# Other region-specific settings...

Helm Chart or Jinja2 Templates:

Use Helm charts to apply overlays and inject region-specific variables.

#### Example
Chart.yaml:
YAML
name: example-app
version: 0.1.0
description: An example application.

Helm Values File Template:

Create a values-template.yaml to dynamically generate values based on region.
YAML
region: {{ .Values.region }}
# Other dynamic values...

Automation and CI/CD Integration:

Integrate the repository with CI/CD pipelines to automate deployments across environments and regions.

#### Closing Summary
By structuring your GitOps repo in a way that separates base configurations from environment-specific overlays and region-specific variables, you can effectively manage 15 unique combinations without duplicating YAML. This approach leverages templating engines like Helm or Jinja2 to inject dynamic values and maintain consistency across environments and regions.

#### Common Mistakes/Follow-Ups

Mistake: Duplicating YAML files for each combination.

#### Follow-up
How would you manage changes if the base configuration needs updates?
Mistake: Overcomplicating overlays with too many specific details in the base config.

#### Follow-up
What are the trade-offs between complexity in the base vs. environment-specific configurations?

**Code:**

```yaml
# Example of common.yaml (base)
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: nginx:latest

# Example of overlay.yaml (environment/dev)
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: nginx:latest

# Example of values.yaml (region/us-east-1)
region: us-east-1
```

