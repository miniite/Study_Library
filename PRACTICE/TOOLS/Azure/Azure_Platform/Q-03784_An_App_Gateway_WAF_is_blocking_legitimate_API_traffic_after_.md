---
type: "practice"
difficulty: "Deep"
---

# An App Gateway WAF is blocking legitimate API traffic after a rule update; how do you tune managed rule sets, exclusions, and detection vs prevention mode?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Networking and Global Traffic](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/Azure_Networking_and_Global_Traffic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To diagnose and resolve an issue where an Azure App Gateway Web Application Firewall (WAF) is blocking legitimate API traffic post a rule update, you'll need to carefully tune managed rules, configure exclusions, and understand the detection vs prevention modes. The key steps involve analyzing logs, identifying problematic rules, adjusting thresholds or customizing rules, applying exclusions for known good requests, and balancing between security and availability.

#### Triaging the Question

The core of this question is about managing WAF settings in Azure App Gateway to ensure that legitimate API traffic isn't inadvertently blocked. The interviewer is probing your understanding of how managed rule sets work, the importance of setting up proper exclusions, and the balance between detection and prevention modes in a WAF.

#### Assumptions
Rule Update: Recent changes have been made to the managed ruleset.
Legitimate Traffic: There are documented cases where legitimate API requests are being blocked.
Logging: You can access detailed logs from App Gateway, including WAF logs and application logs.
Configuration Tools: The Azure portal or REST APIs are available for modifying settings.

#### First-Principles Solution

Review Logs:

WAF Logs: Check the WAF logs in Application Insights or Event Grid to identify which rules triggered blocks on legitimate traffic. Focus on identifying patterns, such as specific time periods or types of requests.
Application Logs: Inspect application logs for any anomalies that correlate with blocked requests.

Analyze Managed Rules:

Managed Rule Set Configuration: Review the configuration of the managed rule set to see which rules are enabled and their default settings.
Rule Details: For each rule, understand its intended use case (e.g., SQL injection prevention) and its impact on legitimate traffic.

Adjust Detection vs Prevention Mode:

Detection Only Mode: Switching a rule to detection mode allows you to monitor its effectiveness without blocking traffic. This is useful for validating rules before making changes.
Tune Rule Thresholds: If a rule is too aggressive, adjust the threshold or tune it to exclude legitimate traffic patterns.

Configure Exclusions:

IP Address Exclusion: Exclude known good IP addresses from being blocked by specific rules using the "Advanced Security" feature in App Gateway.
URL Path Exclusions: Create exclusions for specific URL paths that are frequently accessed and should not be blocked by certain rules.

Customize Rules:

Rule Customization: Modify existing managed rules to better fit your application's needs, such as adding custom signatures or adjusting parameters.
Add Custom Rules: If a specific pattern is causing issues, add a custom rule that precisely targets the issue while minimizing false positives.

Test and Validate:

After making changes, thoroughly test the application to ensure that legitimate traffic flows unimpeded and that no new issues arise.
Monitor logs continuously to catch any unexpected behavior or new attack vectors.

#### Closing Summary

By systematically analyzing logs, tuning managed rules, configuring exclusions, and balancing detection vs prevention modes, you can effectively resolve WAF blocking legitimate API traffic. This approach ensures a high level of security while maintaining application availability.

#### Common Mistakes/Follow-Ups
Over-Reliance on Detection Only Mode: While useful for validation, it should not be used in production without careful consideration.
Excessive Exclusions: Overly broad exclusions can undermine the overall security posture.
Failure to Monitor Post-Tuning: Continuous monitoring is crucial to catch any new issues that may arise due to changes.

Would you like me to elaborate on any of these points further, or do you have a specific scenario in mind where I could provide more detail?

**Comparison:**

```
Rule ID	Name	Status	Match Variable	Operator	Negation Condition
942010	SQL Injection Prevention	Enabled	HTTPRequest.UserAgent	NotContains	True
942100	Cross-Site Scripting (XSS) Prevention	Enabled	HTTPRequest.HttpMethod	Equals	False
```

**Code:**

```python
# Example Python code for setting WAF exclusions using Azure SDK
from azure.mgmt.web import WebSiteManagementClient
from azure.common.credentials import ServicePrincipalCredentials

credentials = ServicePrincipalCredentials(client_id='your-client-id', secret='your-client-secret', tenant='your-tenant-id')
web_client = WebSiteManagementClient(credentials, 'your-subscription-id')

# Adding an IP exclusion
exclusion_params = {
    "properties": {
        "resourceId": "/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.Web/sites/app-gateway",
        "action": "Allow",
        "value": ["192.0.2.1"]
    }
}
web_client.web_apps.add_ip_restriction(exclusion_params)
```

**Example input / output:**

```text
Input (HTTP Request):
- URL: /api/v1/user/profile
- Method: GET

Output (WAF Log):
- Rule ID: 942010, SQL Injection Prevention
- Action Taken: Block
- Reason: User-Agent header contains suspicious characters

Expected Output:
- Switch rule to detection mode and monitor logs for a week.
- If no false positives, set it back to prevention with adjusted threshold.
```

**Data:**

```json
{
  "ruleSetId": "/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.Web/applicationGateways/app-gateway-name/ruleSets/default-rule-set",
  "rules": [
    {
      "name": "SQL Injection Prevention",
      "status": "Enabled",
      "parameters": {
        "matchVariable": "HTTPRequest.UserAgent",
        "operator": "NotContains",
        "negationCondition": true,
        "selectionVariable": "",
        "selectionMatchValue": ""
      }
    },
    {
      "name": "Cross-Site Scripting (XSS) Prevention",
      "status": "Enabled",
      "parameters": {
        "matchVariable": "HTTPRequest.HttpMethod",
        "operator": "Equals",
        "negationCondition": false,
        "selectionVariable": "",
        "selectionMatchValue": ""
      }
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Analyze Logs
4. Is the issue clear?
5. Tune Rule Thresholds
6. Identify Problematic Rules
7. Adjust Rule Parameters
8. Add or Modify Exclusions
9. Test and Validate Changes
10. Monitor Continuously

