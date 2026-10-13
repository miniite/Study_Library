---
type: "practice"
difficulty: "Deep"
---

# Walk me through designing a 'secrets scorecard' per service that grades static-secret usage, TTL hygiene, rotation coverage, and audit completeness. How do you drive it into engineering OKRs without it becoming shelfware?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Secret Rotation](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Shared_Secret_Rotation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Designing a 'secrets scorecard' involves creating a comprehensive metrics dashboard that grades services based on their secrets management practices, such as static-secret usage, time-to-live (TTL) hygiene, rotation coverage, and audit completeness. To ensure this system is effective, we integrate it with the engineering OKRs and use it for continuous improvement.

#### Triaging the Question

The question revolves around designing a metrics-based assessment tool to evaluate secrets management practices across services. The key aspects are:

Static-secret usage: Ensuring secrets are not hardcoded in code or configuration.
Time-to-live (TTL) hygiene: Monitoring secret expiration and ensuring timely rotation.
Rotation coverage: Verifying that all critical secrets have defined and automated rotation mechanisms.
Audit completeness: Confirming thorough logging and monitoring of secret accesses.
Step-by-Step Solution

Define Metrics:

Static-Secret Usage: Develop a script to scan code repositories for hardcoded secrets using tools like bandit or secrets.
TTL Hygiene: Set up a cron job or continuous integration (CI) pipeline to check secret expiration dates and trigger alerts.
Rotation Coverage: Integrate with secret management systems like HashiCorp Vault to verify that rotation policies are in place and enforced.
Audit Completeness: Implement logging and monitoring to track all access logs, ensuring there's a clear audit trail.

Implement Continuous Monitoring:

Use Kubernetes (K8s) native secrets management capabilities or third-party tools like kubeseal for secure storage of secrets in environments.
Deploy agent-based collectors or integrate with cloud-native observability tools to monitor secret usage and access patterns.

Build the Scorecard:

Design a dashboard using tools like Grafana, Prometheus, or custom web applications built on frameworks like React or Angular.
Display metrics for each service in a clear, actionable format: green for good practices, amber for warnings, and red for critical issues.

Integrate with Engineering OKRs:

Define OKR KPIs based on the scorecard metrics.

#### Example
OKRs: "Increase static-secret usage detection rate by 50% in Q1" or "Reduce unrotated secrets by 30% by year-end."
Ensure alignment with broader DevOps goals, such as security and compliance.

#### Closing Summary

By creating a comprehensive secrets scorecard, we can drive continuous improvement in secret management practices. Integrating these metrics into the engineering OKRs ensures that they are not just shelfware but integral to the development process.

#### Common Mistakes
and Follow-Ups
Mistake: Focusing only on static-secret usage without addressing dynamic secrets or third-party integrations.

#### Follow-Up
Discuss how to extend the scorecard to cover other aspects of secrets management, such as encryption at rest and in transit.

**Comparison:**

```
Service	Static Secret Usage	TTL Hygiene	Rotation Coverage	Audit Completeness
api-gateway	Amber (30%)	Green	Red (70%)	Green
```

**Code:**

```python
# Example Python script for detecting hardcoded secrets
import subprocess

def detect_secrets(code_dir):
    result = subprocess.run(['bandit', '-r', code_dir], capture_output=True, text=True)
    return result.stdout

detect_secrets("/path/to/repo")
```

**Example input / output:**

```

Input:

Service: api-gateway
Metrics:
Static Secret Usage: 30% of secrets detected
TTL Hygiene: 95% of secrets have valid TTLs
Rotation Coverage: 70% of secrets are rotated
Audit Completeness: 80% coverage

Output:

PLAINTEXT
Copy
Scorecard for api-gateway:

- Static Secret Usage (amber): 30% detected, improve by 20%
- TTL Hygiene (green): All secrets have valid TTLs
- Rotation Coverage (red): Only 70% of secrets are rotated, increase to 100%
- Audit Completeness (green): Good logging and monitoring in place

Recommended actions:
1. Increase static-secret usage detection rate.
2. Ensure all secrets have defined rotation policies.
3. Review audit logs for potential issues.
```

**Data:**

```yaml
- service: api-gateway
  staticSecretUsage: "amber"
  ttlHygiene: "green"
  rotationCoverage: "red"
  auditCompleteness: "green"
```

**Flow:**

1. Detect Static Secrets
2. Detect TTL Hygiene
3. Check Rotation Coverage
4. Verify Audit Completeness
5. Generate Scorecard

