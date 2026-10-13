---
type: "practice"
difficulty: "Expert"
---

# A self-hosted runner was compromised and could read other repos' secrets; what isolation and hardening would you put in place so one bad job can't pivot across the fleet?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Supply Chain Security and Provenance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/Supply_Chain_Security_and_Provenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a compromised self-hosted runner does not compromise the entire CI/CD pipeline, we need to implement strong isolation and hardening practices. The core idea is to prevent any malicious actor from pivoting across repositories or accessing sensitive secrets. Here’s how we can achieve this:

Isolation: Use dedicated runners for each repository.
Hardening: Apply strict access controls and security measures on the runner infrastructure.
Monitoring: Implement continuous monitoring and alerting to detect any unauthorized activities.

Let's dive into these steps in detail from first principles.

**Comparison:**

| Runner | Repositories |
|--------|--------------|
| runner1 | repo1        |
| runner2 | repo2        |

**Code:**

```python
# Example of a simple function to manage runners per repository
def get_runner_for_repo(repo_name):
    if repo_name not in runner_map:
        raise ValueError(f"Runner for {repo_name} is not configured")
    return runner_map[repo_name]
```

**Example input / output:**

```text
- **Input**: A request to run a job in `repo1`
- **Output**: The runner `runner1` is selected and the job is executed.
```

**Data:**

```json
{
  "runner_map": {
    "repo1": "runner1",
    "repo2": "runner2"
  }
}
```

**Flow:**

1. based on repository
2. CI/CD Request
3. Runner Selection
4. Run Job
5. Maintain Runner Isolation
6. Continuous Monitoring
7. Access Control Enforcement

