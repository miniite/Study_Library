---
type: "practice"
difficulty: "Expert"
---

# How do you architect deploy safety so that the blast radius of a bad config push (which needs no build) is bounded the same way a code deploy is, given config often bypasses the pipeline's staged rollout entirely?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Standard Deployment Strategies](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Standard_Deployment_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the blast radius of a configuration change is managed similarly to how it’s handled in code deployments, we need to architect a robust deployment safety framework. The core idea is to establish a controlled and staged rollout process for configurations just like we do with code, even though configuration changes typically bypass traditional build pipelines. This involves setting up a config validation pipeline, integrating with existing infrastructure, and enforcing strict change control policies.

The question revolves around ensuring that when a bad configuration push occurs (which requires no build), the impact is limited to a predefined set of systems or environments just like in a code deployment. We need to define the blast radius, which includes identifying the affected components, monitoring for failures, and rolling back changes if necessary.

#### Triage
What the Question Is Asking

The interviewer is probing our ability to design a scalable and reliable system that can handle configuration changes without breaking the build process but still maintains safety and control. We need to consider assumptions such as existing infrastructure, the nature of configuration changes (e.g., environment-specific), and potential failure modes.

#### First-Principles Solution
Define the Blast Radius: Identify which environments or services will be affected by a configuration change.
Configuration Validation Pipeline: Create a pipeline that validates configurations before they are applied.
Staged Rollout Mechanism: Implement a staged rollout strategy to roll out changes incrementally, monitoring each stage for issues.
Automated Monitoring and Fail-Safes: Set up automated checks and fail-safes to detect and mitigate any issues during the deployment process.
Rollback Strategy: Have a well-defined rollback plan in place should something go wrong.
Step-by-Step Solution

Identify Affected Services and Environments:

Define the scope of each configuration change (e.g., specific environments like staging, production).
Use service discovery to automatically identify dependent services that need to be checked for compatibility.

Create a Config Validation Pipeline:

Develop a pipeline that validates configurations against known good practices and business rules.
This can include static code analysis tools, security checks, and integration tests specific to the environment.

Staged Rollout Mechanism:

Implement a mechanism for rolling out changes in stages (e.g., from staging to pre-production).
Use load balancers or service mesh features to route traffic appropriately during the rollout.
Monitor each stage using observability tools to detect issues early.

Automated Monitoring and Fail-Safes:

Set up automated monitoring for critical metrics like error rates, latency, and availability.
Integrate with alerting systems to notify stakeholders of potential issues.
Implement auto-healing or self-correction mechanisms if possible (e.g., service restarts).

Rollback Strategy:

Ensure that the rollback plan is tested regularly and can be executed quickly if necessary.
Use version control for configurations to easily revert to a known good state.

#### Closing Summary

By implementing these steps, we can ensure that configuration changes are handled with the same rigor as code deployments. This includes defining the blast radius, validating configurations, staging rollouts, monitoring for issues, and having robust rollback plans in place. The key is to make sure that even though there’s no build step, every other aspect of the deployment process remains controlled and reliable.

#### Common Mistakes/Follow-Ups
Failing to Enforce Validation: Not integrating a validation pipeline can lead to unvetted configurations being deployed.
Insufficient Monitoring: Lack of real-time monitoring can result in undetected issues causing significant disruptions.
Inadequate Rollback Plan: Without a clear rollback strategy, recovering from failures can be challenging and time-consuming.

**Comparison:**

```
Stage	Monitoring Metric	Alert Thresholds
Staging	Error Rate	>2%
	Latency	>10ms
Production	Availability	<95%
```

**Code:**

```python
# Example of a basic validation pipeline step
def validate_configuration(config):
    # Check against known good practices
    if not check_security_rules(config):
        raise ValueError("Configuration failed security checks")
    return True

def check_security_rules(config):
    # Implement detailed security checks here
    pass
```

**Example input / output:**

```text
- **Input**: A new configuration file for the `staging` environment.
- **Output**:
  - Validation pipeline runs and identifies a potential security issue.
  - Rollout to staging is paused, issues are resolved.
  - Change is approved and successfully deployed to production after thorough testing.
```

**Data:**

```json
{
  "environments": ["staging", "production"],
  "affected_services": {
    "staging": ["serviceA", "serviceB"],
    "production": ["serviceC"]
  }
}
```

**Flow:**

1. Pass
2. Fail
3. Start Deployment
4. Validate Config
5. Staging Rollout
6. Reject Changes
7. Monitor Stages
8. Rollback if Issues Detected
9. Complete Deployment

