---
type: "practice"
difficulty: "Intermediate"
---

# How would you enforce YAML style and safety across dozens of repos in CI, and what specific yamllint rules matter most for Kubernetes and Ansible content?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Linting YAML](<../../../../TOOLS/YAML/YAML_Core/Linting_YAML.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Enforcing YAML style and safety in a DevOps environment with multiple repositories is crucial to maintain consistency and prevent runtime errors. For intermediate experience, we need to focus on integrating yamllint checks into our CI/CD pipelines and understanding the specific rules that are critical for Kubernetes and Ansible content.

#### Triage

The question asks how to enforce YAML style and safety across multiple repositories in a CI environment. It also seeks to identify the most important yamllint rules for Kubernetes and Ansible content. The key is to integrate static code analysis tools, understand rule priorities, and ensure consistent application of these rules.

Step-by-Step Solution
Integrate yamllint into CI/CD Pipelines:
Use a CI tool like Jenkins, GitLab CI, or GitHub Actions to run yamllint as part of the build process.
Define and Prioritize YAML Rules:
Kubernetes: Focus on rules that ensure valid syntax, correct structure, and security.
Ansible: Concentrate on rules related to variable interpolation, role existence, and best practices.

#### First-Principles Solution

Setup CI/CD Integration:

YAML
# Example GitLab CI configuration
stages:
  - lint

yamllint-check:
  stage: lint
  script:
    - pip install yamllint
    - yamllint .

Define yamllint Rules for Kubernetes:

Syntax check to ensure the YAML file is well-formed.
Schema validation using Kubernetes-specific schemas like kubeval.
Security checks such as disallowing apiVersion and kind from being modified in templates.

Define yamllint Rules for Ansible:

Variable interpolation to ensure variables are correctly used.
Role existence checks to prevent references to non-existent roles or tasks.
Best practice rules like disallowing hard-coded paths and ensuring consistent naming conventions.

#### Closing Summary

By integrating yamllint into our CI/CD pipelines, we can enforce consistent YAML style and safety across multiple repositories. For Kubernetes, focus on syntax validation, schema checks, and security measures. For Ansible, prioritize variable interpolation, role existence, and best practices to ensure robust and maintainable configurations.

#### Common Mistakes
Overlooking Security Rules: Failing to include rules that protect against potential injection attacks.
Ignoring Best Practices: Dismissing rules related to consistent naming conventions or variable usage patterns.
Incomplete Rule Sets: Not covering all critical aspects of each content type (Kubernetes, Ansible).

**Code:**

```yaml
# Example GitLab CI configuration
stages:
  - lint

yamllint-check:
  stage: lint
  script:
    - pip install yamllint kubeval
    - yamllint --strict .
```

**Example input / output:**

```text
# Sample Kubernetes YAML (valid)
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: nginx
      image: nginx

# Sample Ansible YAML (valid)
---
- hosts: all
  tasks:
    - name: Install Nginx
      yum:
        name: nginx
```

**Flow:**

1. CI/CD Integration
2. Integrate yamllint
3. Define and Prioritize Rules
4. Kubernetes - Syntax, Schema,Security
5. Ansible - VariableInterpolation, Role Existence,Best Practices
6. Run in CI Pipeline

