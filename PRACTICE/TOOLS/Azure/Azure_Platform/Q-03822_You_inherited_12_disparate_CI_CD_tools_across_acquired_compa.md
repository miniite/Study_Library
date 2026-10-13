---
type: "practice"
difficulty: "Expert"
---

# You inherited 12 disparate CI/CD tools across acquired companies; lay out a multi-year consolidation strategy toward one platform on Azure DevOps or GitHub, and how you avoid a mutiny.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Azure Pipelines and Release Approvals](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/Azure_Pipelines_and_Release_Approvals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When consolidating multiple CI/CD systems onto a single platform like Azure DevOps or GitHub, we face significant challenges in ensuring seamless integration while maintaining team morale and productivity. Our strategy will focus on gradual migration, leveraging automation tools, and providing comprehensive training to ease the transition.

The core idea is to develop a phased approach that minimizes disruption by gradually moving each company’s CI/CD processes to the new platform. We’ll use Azure DevOps as our target since it offers extensive features tailored for complex enterprises. The plan will involve several key steps:

Assessment and Planning: Inventory all existing tools, document their workflows, and assess compatibility with Azure DevOps.
Pilot Migrations: Select a small team from each acquired company to pilot the new platform in parallel environments.
Tool Integration: Develop scripts or use third-party tools to automate the migration of pipelines, repositories, and other CI/CD artifacts.
Comprehensive Training: Offer extensive training sessions for developers and DevOps engineers to ensure they are proficient with the new system.
Rollout: Gradually roll out the platform across teams, providing support and addressing any issues that arise.

**Code:**

```python
# Example of a Python script for migration
import os
from azure.devops.connection import Connection

def migrate_repos(repo_info):
    personal_access_token = os.getenv('AZURE_DEVOPS_PAT')
    connection = Connection(base_url='https://dev.azure.com/your_org', creds=personal_access_token)

    project_client = connection.clients.get_project_client(project=repo_info['project_name'])

    for repo in repo_info['repos']:
        print(f"Migration: {repo}")
        # Perform actual migration logic here
```

**Example input / output:**

```text
# Input - Initial CI/CD configuration for companyA
- Repositories: repo1, repo2
- Pipelines: pipeline1, pipeline2

# Output - Migrated configurations to Azure DevOps
- Repositories: successfully migrated repo1 and repo2
- Pipelines: successfully migrated pipeline1 and pipeline2
```

**Data:**

```json
{
  "companyA": {
    "repos": ["repo1", "repo2"],
    "pipelines": ["pipeline1", "pipeline2"]
  },
  "companyB": {
    "repos": ["repo3", "repo4"],
    "pipelines": ["pipeline3", "pipeline4"]
  }
}
```

