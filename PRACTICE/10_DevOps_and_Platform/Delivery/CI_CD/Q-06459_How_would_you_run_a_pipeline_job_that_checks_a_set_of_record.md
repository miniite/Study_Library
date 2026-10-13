---
type: "practice"
difficulty: "Intermediate"
---

# How would you run a pipeline job that checks a set of recording rules can be loaded, without actually pushing them to production?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Validating Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this CI/CD question, we'll develop an automated pipeline that verifies recording rules are correctly configured and valid before deploying them to production. This involves writing a script or using a tool to check if the rules can be loaded without errors in a safe environment.

#### Triage

The core of this problem is to ensure the integrity and correctness of configuration changes in a CI/CD pipeline, specifically for monitoring systems like Prometheus. The key aspects are:

Understanding the Recording Rules: Each rule must follow syntactic and semantic validity checks.
Environment Simulation: The job should run these rules on a simulated or staging environment to mimic production without actual impacts.
Error Handling: Properly logging and handling any issues detected during this validation process.

#### First Principles Solution
Configuration Parsing: Read the recording rules from configuration files (e.g., YAML).
Rule Validation: Use built-in tools or custom scripts to validate these rules against known schemas and formats.
Staging Environment Setup: Run the validated rules in a non-production environment that mimics production.
Error Reporting: Log any issues found during validation for easy debugging and fixing.
Step-by-Step Solution
Define Validation Logic:
Use a configuration parser to read the recording rules from a file (e.g., prometheus_rules.yaml).
Implement or use an existing library to validate these rules against the Prometheus rule format specification.
Staging Environment Initialization:
Set up a staging environment that mirrors production settings, including network configurations and data feeds.
Run Validation:
Load the validated rules into the staging environment using a tool like promtool.
Capture any errors or warnings output by promtool during this process.
Post-Validation Actions:
Log all issues detected in the validation step.
If no issues, approve for production deployment; otherwise, correct and revalidate.

#### Implementation

Here is a simplified example of how you might implement this:

PYTHON
def validate_prometheus_rules(rules_path):
    import yaml
    from prometheus_client.parser import text_string_to_metric_family

    # Parse the rule file
    with open(rules_path) as f:
        rules = yaml.safe_load(f)

    errors = []

    for group in rules.get("groups", []):
        for rule in group["rules"]:
            try:
                _ = text_string_to_metric_family(rule["expr"])
            except Exception as e:
                errors.append({"rule": rule, "error": str(e)})

    return errors
Staging Environment Setup (Example)
YAML
# Prometheus configuration file (prometheus.yml)
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'staging'
    static_configs:
      - targets: ['staging-monitoring-service']
Dry-Run Example

Suppose prometheus_rules.yaml contains the following rules:

YAML
groups:
  - name: example_group
    rules:
      - record: cpu_usage
        expr: rate(container_cpu_usage_seconds_total[1m])

Running the validation script might produce no errors, or it could log an error if there was a syntax issue.

#### Closing Summary

This approach ensures that recording rules are valid and will load without issues in production. It is crucial for maintaining system integrity and avoiding potential outages due to misconfiguration. The key steps involve parsing, validating, running the rules in a safe environment, and logging any errors detected.

#### Common Mistakes/Follow-Ups
Not Covering All Validation Aspects: Ensure that all aspects of rule validation are covered (syntax, semantics).
Ignoring Staging Environment Differences: Make sure the staging environment closely matches production to avoid issues during actual deployment.
Failure to Log and Act on Errors: Properly logging errors and having a process in place for addressing them is crucial.

**Comparison:**

```
Rule	Error
rate(container_cpu_usage_seconds_total[1m])	None
```

**Code:**

```python
def validate_prometheus_rules(rules_path):
    import yaml
    from prometheus_client.parser import text_string_to_metric_family

    # Parse the rule file
    with open(rules_path) as f:
        rules = yaml.safe_load(f)

    errors = []

    for group in rules.get("groups", []):
        for rule in group["rules"]:
            try:
                _ = text_string_to_metric_family(rule["expr"])
            except Exception as e:
                errors.append({"rule": rule, "error": str(e)})

    return errors
```

**Example input / output:**

```

Running the validation script on prometheus_rules.yaml:

Input:

YAML
Copy
groups:
  - name: example_group
    rules:
      - record: cpu_usage
        expr: rate(container_cpu_usage_seconds_total[1m])

Output: No errors logged.
```

**Data:**

```yaml
# Example prometheus_rules.yaml
groups:
  - name: example_group
    rules:
      - record: cpu_usage
        expr: rate(container_cpu_usage_seconds_total[1m])
```

**Flow:**

1. No Errors
2. No Issues
3. Issues Found
4. Read Rules
5. Parse YAML
6. Validate Syntax
7. Staging Environment Setup
8. Run Rules in Staging
9. Approval for Production
10. Log and Fix

