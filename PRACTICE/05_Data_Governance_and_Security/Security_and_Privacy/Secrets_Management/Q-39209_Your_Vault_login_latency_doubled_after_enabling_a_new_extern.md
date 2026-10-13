---
type: "practice"
difficulty: "Deep"
---

# Your Vault login latency doubled after enabling a new external OIDC provider. How do you troubleshoot the auth path?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Vault Outage Resilience](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Vault_Outage_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When faced with an unexpected increase in login latency after enabling an external OIDC provider, our goal is to systematically diagnose the authentication path from user interaction to Vault response. Key steps include examining network latency, configuration settings, and potential bottlenecks within both the OIDC setup and Vault itself.

#### Triage
Understanding the Issue

The primary issue here is a sudden doubling of login latency after enabling an external OIDC provider. This increase suggests that there are likely one or more points along the authentication path where performance has degraded. We need to determine if this degradation is due to network issues, configuration missteps, or internal Vault processing overhead.

#### Assumptions
The OIDC setup and Vault configurations were functioning correctly before enabling the new provider.
No significant changes were made to the network infrastructure during this period.
There are no known service disruptions affecting either the OIDC provider or Vault during the incident.
Step-by-Step Troubleshooting
1. Network Latency Analysis

First, we need to understand if increased latency is due to network issues. Use traceroute and ping commands from a client machine attempting to authenticate to identify where delays occur.

BASH
# Example command: traceroute vault-server

This will show the path packets take to reach Vault and any potential delay points.

2. Check OIDC Provider Configuration

Examine the OIDC provider's configuration for any changes or misconfigurations that might affect performance. Verify:

The discovery URL is correct.
Client ID, client secret, and other credentials are correctly set.
The OIDC provider's metadata endpoint is reachable and responding quickly.
3. Vault Configuration Review

Review Vault’s OIDC setup to ensure it is configured optimally:

Check the auth backend configuration for any performance tuning options that might have been overlooked.
Ensure the token TTL settings are appropriate, as overly short timeouts can cause frequent requests.
HCL
# Example Vault HCL configuration snippet
oidc_auth_backend {
  ttl = "10m"
}
4. Performance Monitoring

Use monitoring tools like Prometheus and Grafana to track performance metrics such as request latency, error rates, and resource usage:

Monitor the auth backend performance specifically for OIDC requests.
Check Vault’s CPU and memory usage during peak authentication times.
5. Logs Analysis

Examine both Vault logs and OIDC provider logs to look for any errors or warnings that might indicate issues:

BASH
# Example log analysis commands
tail -f /var/log/vault/auth/oidc.log

Check for error messages, slow queries, or failed authentications.

6. Load Testing

Conduct load testing using tools like k6 or LoadRunner to simulate user authentication and measure performance under stress:

BASH
# Example k6 script snippet
import http from 'k6/http';
import { check } from 'k6';

export default function() {
    let res = http.get('https://vault-server/auth/oidc/login');
    check(res, { "status was 200": (r) => r.status === 200 });
}

#### Closing Summary
+ Common Mistakes

By systematically analyzing the network path, configuration settings, and performance metrics, we can identify the root cause of increased latency. Common mistakes include:

Misconfigured OIDC provider settings.
Overlooked Vault backend tuning options.
Insufficient monitoring leading to delayed detection of issues.

#### Follow-Up
Questions
How would you optimize the OIDC provider’s setup for high-performance authentication?
What are some best practices for maintaining and scaling Vault instances with external IDP integrations?

**Comparison:**

```
Metric	Value
Request Latency (min)	50ms
Error Rate (%)	0.1%
CPU Usage (%)	75%
Memory Usage (MB)	4000
```

**Code:**

```hcl
# Example HCL configuration snippet
oidc_auth_backend {
  ttl = "10m"
}
```

**Example input / output:**

```text
# Example input and output for k6 load testing script
$ k6 run -t 30s test.js
[info] [2023-09-28T14:57:21.00Z] [INFO] [k6,cmd] Starting execution: running 1 vU, 1 threads for 30s
[info] [2023-09-28T14:57:21.00Z] [INFO] [test.js] GET https://vault-server/auth/oidc/login - status code 200, duration 20ms
```

**Data:**

```json
{
  "user": "admin",
  "method": "OIDC",
  "token_ttl": "10m"
}
```

**Flow:**

1. User Initiates Login
2. OIDC Provider Authentication
3. Vault Auth Backend Validation
4. Vault Issues Token
5. User Access Granted

