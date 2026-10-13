---
type: "practice"
difficulty: "Intermediate"
---

# Your alerting pipeline deploys through CI but the on-call escalation config lives in a SaaS UI; how would you bring that config under version control and pipeline review?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Deployment Observability](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of having an on-call escalation configuration live outside our CI/CD pipeline, we need to ensure consistency, traceability, and automation in its deployment. By bringing this configuration under version control and integrating it with the CI/CD process, we can leverage modern DevOps practices to maintain robust alerting and improve on-call readiness.

#### Triage
Understanding the Question

The core of the question is about aligning an existing, non-version-controlled configuration (on-call escalation) with a fully automated CI/CD pipeline. The key points being probed include:

Integration of manual processes: How we can automate the process of updating and deploying on-call configurations.
Version control practices: Ensuring that changes are tracked and can be rolled back if necessary.
CI/CD integration: Automating the deployment of both code and configuration through the same pipeline.

#### Assumptions
The current on-call escalation config is stored in a SaaS application's UI, without any direct export or version control options.
We have access to scripting capabilities within our CI/CD platform (e.g., Jenkins).
The alerting system and on-call configurations can be represented as JSON/YAML files.

#### Solution
Identify the Current Configuration Format: Determine how the on-call escalation config is structured in the SaaS UI. This might involve scraping data or using API endpoints if available.
Develop a Script to Extract Configurations: Write a script that can export the current state of the on-call configurations from the SaaS application and save it as a version-controlled file (e.g., JSON, YAML).
Integrate the Script into CI/CD Pipeline:
Add a step in the CI pipeline to run the script automatically after code changes.
Ensure that any new or updated on-call escalations are reflected in this configuration file.
Step-by-Step Solution
Export Configurations: Develop a script (in Python, Bash, etc.) that connects to the SaaS application's API and exports the current state of the on-call escalation configurations. Store these as version-controlled files.
Automate Script Execution:
Add this script step in your CI/CD pipeline after code deployment steps.
Integrate with Version Control: Commit and push these configuration files to a Git repository (e.g., GitHub, Bitbucket).
Review Process: Ensure that the changes are reviewed before being deployed through the CI/CD process.
Rollback Mechanism: Implement a rollback mechanism in case of issues.

#### Closing Summary

By bringing the on-call escalation configurations under version control and integrating them with our existing CI/CD pipeline, we can ensure that all critical alerting and on-call configurations are tracked, reviewed, and deployed consistently. This approach enhances traceability, minimizes human error, and ensures that our alerting system remains robust.

#### Common Mistakes/Follow-Ups
Not Considering Rollback Mechanisms: Ensure there is a clear rollback process in place.
Overlooking Security Concerns: Ensure sensitive data is handled securely, especially when exporting from the SaaS application.
Manual Script Maintenance: Regularly update and maintain the script to handle any changes in the configuration structure or API.

**Comparison:**

```
Step	Action
1	Identify current configuration format.
2	Develop script for exporting configurations.
3	Integrate script into CI/CD pipeline steps.
4	Commit and push to version control system.
5	Implement rollback mechanism.
```

**Code:**

```python
import requests
import json

# Example Python script for extracting on-call escalation config
def export_on_call_config(api_url, auth_token):
    headers = {"Authorization": f"Bearer {auth_token}"}
    response = requests.get(api_url, headers=headers)
    if response.status_code == 200:
        config_data = response.json()
        with open("on_call_escalation_config.yaml", "w") as file:
            yaml.dump(config_data, file)
        return True
    else:
        print(f"Failed to fetch data: {response.status_code}")
        return False

if __name__ == "__main__":
    api_url = "<YOUR_API_URL>"
    auth_token = "<YOUR_AUTH_TOKEN>"
    export_on_call_config(api_url, auth_token)
```

**Example input / output:**

```text
Input:
API URL: https://example-api.com/on-call-config
Auth Token: <YOUR_AUTH_TOKEN>

Output:
Successfully exported on_call_escalation_config.yaml to Git repository.
```

**Data:**

```yaml
# Example on-call escalation config YAML file
escalations:
  - name: "Team A On-Call"
    email: "team_a@example.com"
    phone: "+1234567890"
  - name: "Team B On-Call"
    email: "team_b@example.com"
    phone: "+9876543210"
```

**Flow:**

1. Identify current configurationformat
2. Develop script for exportingconfigurations
3. Integrate script into CI/CDpipeline steps
4. Commit and push to versioncontrol system
5. Implement rollbackmechanism

