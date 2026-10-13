---
type: "practice"
difficulty: "Deep"
---

# Your self-hosted runners hold cloud credentials and a compromised dependency in one job could exfiltrate them; how would you scope per-job permissions and isolate runners so one bad job's blast radius is minimal?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Supply Chain Security and Provenance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/Supply_Chain_Security_and_Provenance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that our self-hosted GitHub Actions runners are isolated from each other in such a way that one compromised runner does not have access to sensitive credentials of another job or even affect the entire pipeline. This involves scoping per-job permissions and ensuring that each job runs in a controlled environment with minimal blast radius.

First, let’s clarify what we're dealing with: self-hosted runners are running on machines that are managed by us. They can hold cloud credentials (e.g., AWS IAM roles or API keys) and might be compromised if not properly isolated. A single job containing an exfiltrated dependency could expose these credentials to malicious actors.

#### Triage

The question is probing our ability to manage security in a CI/CD pipeline, specifically focusing on the isolation of jobs running on self-hosted runners. We need to ensure that:

Each job operates with minimal privileges.
Credentials are securely stored and accessed only by necessary components.
Compromised jobs do not affect other jobs or the overall infrastructure.

#### First-Principles Solution

To achieve this, we can employ several strategies:

1. Job Isolation via Runner Groups

We can create runner groups for different types of jobs. For instance, one group could be dedicated to critical tasks like deploying sensitive services, while another might handle less critical tasks such as documentation updates.

No Impact
No Impact
Critical Job Group
Credential Exfiltration
No Compromised Credentials
Less Critical Job Group
Normal Operation
2. Runners with Minimal Privileges

Ensure that runners only have the minimum permissions needed to execute their specific tasks. For example, a runner used for deploying a web application might only need access to deployment environments but not sensitive data stores.

YAML
# Example GitHub Actions configuration snippet for minimal privilege jobs
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Application
        run: |
          # Deployment logic here
3. Environment Variables and Secrets Management

Store sensitive information in encrypted environment variables or secrets management tools like GitHub’s built-in secret store, Hashicorp Vault, or AWS Secrets Manager.

BASH
# Example of using a secret variable in a job
steps:
  - name: Set Environment Variable
    run: |
      export CLOUD_CREDENTIALS=$(echo ${{ secrets.CLOUD_CREDENTIALS }} | base64 --decode)
4. Job-to-Runner Binding

Bind jobs to specific runners or groups of runners that have predefined roles. This ensures that only the necessary infrastructure is exposed.

YAML
# Example binding job to a runner with specific role
jobs:
  deploy:
    runs-on: self-hosted
    needs: setup
    environment: production
    strategy:
      matrix:
        runner-role: [web-deploy]
5. Network Segmentation

Ensure that runners and their associated services are isolated in different network segments. This limits the potential damage if a service is compromised.

No Access
Runner
Isolated Network Segment
Critical Services

#### Closing Summary

By implementing these strategies, we can significantly reduce the risk of a single job compromise affecting other parts of our CI/CD pipeline. Each job should operate in a controlled environment with minimal permissions and be isolated from critical services.

#### Common Mistakes/Follow-Ups
Failing to Use Minimal Privileges: Running all jobs on the same runner without proper isolation.
Storing Credentials In Plain Text: Failing to use encrypted secrets or secure storage solutions.
Not Binding Jobs to Specific Runners: Allowing any job to run on any available runner, increasing the risk of cross-jobs attacks.

To further improve security, we could consider implementing additional measures like regular audits, continuous monitoring, and automated alerting for suspicious activities.

**Comparison:**

```
Strategy	Description
Runner Groups	Create distinct groups for different job types.
Minimal Privileges	Assign runners the least privilege necessary to execute tasks.
Secrets Management	Use encrypted environment variables and secret stores.
```

**Code:**

```yaml
# Example GitHub Actions configuration snippet
jobs:
  critical-job:
    runs-on: self-hosted
    steps:
      - name: Run Critical Task
        run: |
          # Logic here using environment variables

  non-critical-job:
    runs-on: self-hosted
    steps:
      - name: Run Non-Critical Task
        run: |
          # Logic here with minimal permissions
```

**Example input / output:**

```text
Input:
- Job A (critical) and Job B (non-critical)
- Compromised credentials in Job A

Output:
- Job B executes normally, isolated from compromised credentials
- No impact on other jobs or pipeline integrity
```

**Data:**

```json
{
  "runner-groups": {
    "web-deploy": {
      "description": "Runner group for deploying web applications",
      "permissions": ["read", "write"],
      "environment": "production"
    },
    "admin-task": {
      "description": "Runner group for administrative tasks",
      "permissions": ["admin"]
    }
  }
}
```

**Flow:**

1. No Impact
2. Restricts Access
3. Job A (Critical)
4. Compromise
5. Exfiltrate Credentials
6. No Compromised Credentialsin Other Jobs
7. Isolated Network Segment
8. Job B (Non-Critical)

