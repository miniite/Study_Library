---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up unified service tagging across env service and version to make deploy-versus-baseline comparisons reliable?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ APM and Trace Analytics](<../../../../TOOLS/Datadog/Datadog_Platform/APM_and_Trace_Analytics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To ensure reliable deploy-versus-baseline comparisons, we need a robust system of unified service tagging in Datadog that encompasses environment (env), service name, and version information. This setup will help us track changes accurately across different environments and baseline services effectively.

To achieve this, we'll follow these steps:

Define the Tagging Schema: Clearly outline which tags are necessary for each service.
Implement Tagging in Code: Ensure all deployment processes tag resources appropriately.
Configure Datadog Dashboards: Use these tags to filter and compare data across environments and versions.

#### Triage
The question is asking how to ensure reliable deploy-versus-baseline comparisons by setting up unified service tagging. The interviewer is probing our understanding of tagging best practices, deployment processes, and configuration in Datadog. We'll assume a multi-environment setup (development, staging, production) and that we need to compare service performance across different versions.

Step-by-Step Solution:

Define the Tagging Schema

Environment Tags: env – values could be dev, staging, prod.
Service Name Tags: service_name – unique identifier for each service.
Version Tags: version – semantic versioning (e.g., 1.0.2, 2.0.0).

Implement Tagging in Code

Ensure that all deployment processes include the necessary tagging. For example, using CI/CD pipelines to tag services.
PYTHON
# Example of tagging a service during deployment
import datadog

def deploy_service(service_name, version):
    datadog.tag('env:staging', f'service:{service_name}', f'version:{version}')
    # Deployment logic here

Configure Datadog Dashboards

Create dashboards that filter and compare metrics based on these tags.
Use tags filters in Datadog to segment data by environment, service, and version.

Monitor and Validate

Set up alerts and notifications for any discrepancies or anomalies in the tagging process.
Periodically validate the tagging through manual checks or automated scripts.

#### Closing Summary
and Common Mistakes/Follow-Ups: Properly setting up unified service tagging is crucial for reliable baseline comparisons across different environments. Key pitfalls include missing tags during deployments, inconsistent tag usage, and not configuring Datadog to leverage these tags effectively. Follow-ups might involve discussing specific tools or plugins used in Datadog, or scaling considerations for large-scale deployments.

**Code:**

```python
import datadog

def deploy_service(service_name, version):
    datadog.tag('env:staging', f'service:{service_name}', f'version:{version}')
    # Deployment logic here
```

