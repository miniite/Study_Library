---
type: "practice"
difficulty: "Deep"
---

# What is Crossplane and how does it enable GitOps for cloud resources?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Multi-cluster Ordering](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Multi-cluster_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Crossplane is a powerful tool that integrates with Kubernetes to manage infrastructure as code (IaC), enabling organizations to apply GitOps principles to their cloud resources. At its core, Crossplane acts as an abstraction layer between your GitOps workflows and the underlying cloud providers. Let’s break down how it works and why it's essential for implementing GitOps.

To start, we need to understand what GitOps is: a set of practices that leverages infrastructure as code and continuous delivery tools to streamline operations and improve reliability in deploying applications to production environments. GitOps relies on a single source of truth stored in Git repositories, which ensures consistency across all environments and reduces human error.

Crossplane enables this by allowing you to define cloud resources (like databases, storage volumes, networking components) using Kubernetes Custom Resource Definitions (CRDs). These CRDs are then managed just like any other Kubernetes resource, meaning they can be versioned, reviewed, and deployed through GitOps workflows.

Here’s a step-by-step explanation of how Crossplane fits into the GitOps picture:

Define Resources as Code: You define your cloud resources using Kubernetes CRDs. For example, you might define a Database or VPC resource in YAML files.
Version Control and CI/CD Pipelines: These YAML files are stored in a Git repository (e.g., GitHub). CI/CD pipelines can be used to automatically validate, test, and deploy these resources into your cloud environment.
Sync State with Desired State: Crossplane continuously monitors the state of the cloud resources and ensures it matches the desired state defined in the CRDs. Any discrepancies are automatically reconciled.
Audit and Traceability: Because everything is stored in Git, you have a complete audit trail of changes made to your infrastructure.

Crossplane supports multiple cloud providers (AWS, GCP, Azure) out-of-the-box, making it highly versatile. It also integrates seamlessly with other Kubernetes-native tools like Helm, FluxCD, and Skaffold.

Now, let’s look at some examples:

#### Example
Defining a Database Using Crossplane
YAML
apiVersion: database.crossplane.io/v1alpha1
kind: PostgreSQL
metadata:
  name: example-db
spec:
  forProvider:
    version: "13"

This YAML file defines a PostgreSQL database that can be versioned and deployed using GitOps practices.

#### Example
Workflow with Crossplane
BASH
# Clone the repository containing the CRD definitions
git clone https://github.com/example/crossplane-repo.git

# Update the CRDs in your repository to reflect new configurations
git commit -am "Added new PostgreSQL database"

# Trigger a CI/CD pipeline to deploy these changes to your Kubernetes cluster
ci-deploy-pipeline.sh

Crossplane ensures that the actual state of the cloud resources (e.g., the PostgreSQL instance) matches the desired state defined in the CRD, providing a seamless GitOps experience.

#### Common Mistakes
and Follow-Ups
Misconfiguration: Failing to properly define the CRDs can lead to resource creation or deletion issues.
Lack of Monitoring: Without proper monitoring, discrepancies between the actual and desired states might go unnoticed.
Resource Versioning: Not using GitOps tools effectively for versioning resources can result in inconsistent deployments.

In summary, Crossplane enables GitOps by abstracting cloud resources into Kubernetes CRDs, allowing them to be managed with the same rigor and discipline as application code. This integration ensures consistency, traceability, and automation across your infrastructure, making it a powerful tool for modern DevOps practices.

**Comparison:**

```
Step	Action	Outcome
1	Clone Repository	https://github.com/example/crossplane-repo.git
2	Update CRDs	New database YAML added: postgresql.yaml
3	Commit Changes	git add ., git commit -m "Added new PostgreSQL database"
4	Push to Git	git push origin main
5	Trigger CI/CD	Run ci-deploy-pipeline.sh to deploy changes
```

**Code:**

```yaml
# Example YAML file defining a PostgreSQL database using Crossplane
apiVersion: database.crossplane.io/v1alpha1
kind: PostgreSQL
metadata:
  name: example-db
spec:
  forProvider:
    version: "13"
```

**Example input / output:**

```text
# Clone the repository containing the CRD definitions
git clone https://github.com/example/crossplane-repo.git

# Update the CRDs in your repository to reflect new configurations
echo "apiVersion: database.crossplane.io/v1alpha1\nkind: PostgreSQL\nmetadata:\n  name: example-db\nspec:\n  forProvider:\n    version: '13'" > postgresql.yaml

git add .
git commit -m "Added new PostgreSQL database"
git push origin main

# Trigger a CI/CD pipeline to deploy these changes to your Kubernetes cluster
ci-deploy-pipeline.sh
```

**Data:**

```
Sample CRD Definition
YAML
Copy
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: postgredbs.database.crossplane.io
spec:
  group: database.crossplane.io
  names:
    kind: PostgreSQL
    listKind: PostgreSQLList
    plural: postgredbs
    singular: postgredb
  scope: Namespaced
  versions:
    - name: v1alpha1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                forProvider:
                  type: object
                  properties:
                    version:
                      type: string
```

**Flow:**

1. Define CRDs
2. Store in Git
3. CI/CD Pipeline TriggersDeployment
4. Kubernetes Cluster AppliesChanges
5. Crossplane Reconciles State

