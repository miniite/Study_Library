---
type: "practice"
difficulty: "Deep"
---

# How would you deploy DAGs to Airflow via GitOps (git-sync or object-storage bundle) rather than baking them into the image, and what are the trade-offs at deploy time?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Push vs Pull Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To deploy Directed Acyclic Graphs (DAGs) to Apache Airflow using GitOps, we can leverage tools like git-sync or object storage bundles instead of baking them into Docker images. This approach offers several advantages in terms of flexibility and maintainability. Here’s a step-by-step breakdown:

TL;DR: By deploying DAGs via GitOps, we ensure that the codebase is version-controlled, allowing for easy collaboration, change tracking, and rollback mechanisms. This method also simplifies the deployment process by decoupling the DAGs from the Airflow image, enabling dynamic updates without re-baking images.

#### Triaging the Question
The question primarily revolves around deploying DAGs in an Airflow environment using GitOps principles. We need to consider how git-sync or object storage bundles can facilitate this deployment and discuss the trade-offs involved, particularly at deploy time.

#### First-Principles Solution

Environment Setup: First, set up your infrastructure with a Git repository that contains the DAGs and Airflow configurations.
GitOps Tools: Use tools like git-sync or store the DAGs in an object storage bucket (e.g., S3) accessible by your Airflow cluster. These tools monitor changes in the codebase and automatically update the Airflow environment.
Airflow Configuration: Configure Airflow to pull DAG files from the specified source at startup, ensuring that the latest version is always running.

#### Trade-Offs

Deploy Time Complexity:

#### Pros
Easier to manage updates and rollbacks, no need for image re-baking.

#### Cons
Initial setup might be more complex due to additional configuration required with git-sync or object storage. Monitoring and troubleshooting can also become slightly more intricate.

Operational Cost:

#### Pros
Reduced costs associated with frequent Docker image rebuilds, especially in scenarios where DAG changes are infrequent.

#### Cons
Additional costs for maintaining GitOps tools and possibly increased storage costs if using object storage.

#### Closing Summary
+ Common Mistakes/Follow-Ups: Deploying DAGs via GitOps offers significant benefits in terms of flexibility and maintainability, making it a preferred approach over baking them into Docker images. However, the initial setup and operational complexity must be carefully managed to avoid common pitfalls such as misconfigurations or unforeseen issues with GitOps tools.

**Comparison:**

```
Pros	Cons
Easier code management	Increased setup complexity
Automatic rollbacks	Potential operational complexity
Reduced image rebuilds	Possible increased costs
```

**Code:**

```python
# Example Airflow DAG configuration in airflow.cfg
[core]
dagbag_import_timeout = 30
dags_are_paused_at_creation = False

[api]
airflow_api_connexion_expose_exception = True

[webserver]
base_url = http://localhost:8080
```

**Example input / output:**

```text
Input:
- Git repository or object storage bucket containing DAGs

Output:
- Airflow environment automatically updates with the latest DAG files at startup
- Seamless integration of CI/CD pipelines for continuous deployment and rollback
```

**Data:**

```json
{
    "git-repo": {
        "url": "https://github.com/example/airflow-dags.git",
        "branch": "main"
    },
    "object-storage": {
        "bucket": "dags-bucket",
        "prefix": "dags/"
    }
}
```

**Flow:**

1. Yes
2. No
3. Deploy DAGs via GitOps
4. Is Git repository or objectstorage used?
5. Set up `git-sync` or objectstorage monitoring
6. Pull latest DAG files at Airflowstartup
7. Continue with traditionalimage baking approach

