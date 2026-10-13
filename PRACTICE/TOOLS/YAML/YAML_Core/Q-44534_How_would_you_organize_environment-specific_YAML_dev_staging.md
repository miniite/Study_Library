---
type: "practice"
difficulty: "Deep"
---

# How would you organize environment-specific YAML (dev staging prod) so you avoid duplicating most of the config?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Refactoring Helm Values](<../../../../TOOLS/YAML/YAML_Core/Refactoring_Helm_Values.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To organize environment-specific YAML configurations effectively, ensuring minimal duplication while maintaining clear separation and manageability, we need to focus on leveraging inheritance, conditional logic, and a structured directory layout. This approach ensures that common configurations are shared across environments, reducing maintenance overhead.

#### Triage

The question is essentially asking how to structure your YAML files for different environments (e.g., development, staging, production) in a DevOps context. The interviewer is probing your understanding of best practices for configuration management and the ability to handle environment-specific configurations without duplicating code excessively. Assumptions include having multiple environments with shared but distinct configurations.

#### First-Principles Solution
Directory Layout: Organize YAML files within a directory structure that reflects different stages.
Inheritance: Use inheritance or includes to share common configurations across environments.
Conditional Logic: Implement conditional logic in the configuration itself where necessary, especially for environment-specific overrides.
Step-by-Step Solution

Directory Structure

CODE
config/
├── common.yaml
├── dev/
│   └── custom.yaml
├── staging/
│   └── custom.yaml
└── prod/
    └── custom.yaml

common.yaml

YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: common-config
data:
  setting1: "value1"
  setting2: "value2"

dev/custom.yaml (Inherits from common.yaml)

YAML
---
include: common.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: dev-config
data:
  setting3: "value3"

staging/custom.yaml (Inherits from common.yaml)

YAML
---
include: common.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: staging-config
data:
  # Staging-specific settings can override or extend the shared configuration.
  setting4: "value4"

prod/custom.yaml (Inherits from common.yaml)

YAML
---
include: common.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: prod-config
data:
  # Production-specific settings.
  setting5: "value5"

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Overly complex inheritance hierarchies leading to difficult-to-maintain configurations.
Failing to use conditional logic effectively, resulting in bloated environment-specific files.

#### Follow-Ups

How do you handle secrets management and ensure security?
What tools or best practices would you recommend for version control of these configurations?

**Code:**

```yaml
# common.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: common-config
data:
  setting1: "value1"
  setting2: "value2"

# dev/custom.yaml
---
include: common.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: dev-config
data:
  setting3: "value3"

# staging/custom.yaml
---
include: common.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: staging-config
data:
  setting4: "value4"

# prod/custom.yaml
---
include: common.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: prod-config
data:
  setting5: "value5"
```

