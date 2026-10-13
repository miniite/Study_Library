---
type: "practice"
difficulty: "Basic"
---

# Why is it a good practice to keep YAML files under version control and review them in pull requests?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Linting YAML](<../../../../TOOLS/YAML/YAML_Core/Linting_YAML.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Keeping YAML files under version control (VCS) and reviewing them in pull requests (PRs) is a crucial best practice because it ensures consistency, traceability, and collaboration within teams. This process helps maintain the integrity of configurations and prevents accidental changes that could disrupt system operations.

#### Triaging the Question

The core question here is why YAML files should be managed through version control systems like Git, and how pull requests can further enhance this management by providing a collaborative review mechanism. The key points to consider are:

#### Cons
istency across environments.
Traceability of configuration changes.
Collaboration among team members.
Mitigating risks associated with accidental or unauthorized changes.

#### First Principles Solution

YAML is often used for defining configurations in infrastructure as code (IaC) tools, such as Kubernetes manifests and AWS CloudFormation templates. These files are critical because they control the state of your system's infrastructure. Here’s how keeping YAML under version control benefits your project:

#### Cons
istency Across Environments: By maintaining a single source of truth for your configurations in Git, you ensure that every deployment environment (development, staging, production) uses the same configuration files. This reduces the risk of divergence and makes it easier to manage updates across different environments.

Traceability of Changes: Every change made to a YAML file is recorded in the commit history. When issues arise, you can easily trace back to find when and why changes were made. This historical data is invaluable for troubleshooting and auditing purposes.

Collaboration Among Team Members: Pull requests in VCS allow team members to review each other’s work before it gets merged into the main codebase. This process ensures that everyone is on the same page regarding configuration updates, which can be especially important when making complex or risky changes.

Mitigating Risks of Accidental Changes: Without version control, a single typo in a YAML file could lead to catastrophic failures. By forcing developers to review their changes before they are deployed, pull requests help catch such errors early on, reducing the risk of production issues.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, keeping YAML files under version control and reviewing them through pull requests is essential for maintaining the integrity and reliability of your infrastructure configurations. This practice ensures consistency, traceability, collaboration, and minimizes risks associated with accidental changes.

Some common mistakes to avoid include:

Not using VCS at all or opting for less robust tools.
Failing to implement a proper review process in pull requests.
Ignoring commit messages that provide context for why changes were made.

#### Follow-up
questions might explore specific tools like GitOps, how branching strategies can be applied to YAML files, and the integration of CI/CD pipelines with version control systems.

**Code:**

```yaml
# Example Kubernetes manifest file (nginx.yaml)
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

