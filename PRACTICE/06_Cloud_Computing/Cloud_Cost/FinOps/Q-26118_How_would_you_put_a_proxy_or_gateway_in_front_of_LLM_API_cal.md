---
type: "practice"
difficulty: "Deep"
---

# How would you put a proxy or gateway in front of LLM API calls so you get per-team token spend visibility and can enforce quotas before the monthly bill arrives?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shadow Mode for Cost Policies](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shadow_Mode_for_Cost_Policies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To achieve per-team token spend visibility and enforce quotas for Large Language Model (LLM) API calls, we need to implement a proxy or gateway that intercepts requests, logs and aggregates usage metrics by team, enforces predefined quotas, and reports back on the monthly spend. This solution leverages a combination of Kubernetes, Istio service mesh, and custom metrics pipelines.

#### Triaging the Question

The interviewer is probing your understanding of cloud cost governance, particularly in the context of LLM APIs, which are often charged by token usage. The key aspects to address include:

Visibility: Ensuring each team can see its own token consumption.
Quota Enforcement: Setting and enforcing limits on API calls per team before they incur costs.
Reporting: Providing accurate and timely reports on the monthly spend.

#### Assumptions
You have a Kubernetes cluster where your LLM services are deployed.
The LLM API is stateless and request-driven, making it suitable for proxying through an Istio service mesh.
You use a cloud provider that supports custom cost tags or annotations to track usage by teams.

#### First-Principles Solution
Step 1: Design the Proxy/Gateway

We will deploy a proxy/gateway in front of our LLM services using Kubernetes and Istio for advanced networking capabilities. This setup allows us to intercept requests, apply rules, and log necessary metadata such as team identifiers.

BASH
# Deploy Istio Service Mesh on Kubernetes
kubectl apply -f https://istio.io/downloadIstio/istio-1.12.3/install.yaml --set values.global.meshExpansionMode=automatic

# Apply a sample gateway configuration
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: llm-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
Step 2: Define Custom Telemetry and Quotas

We will configure custom telemetry to track LLM API calls and set up quotas per team. Istio can be used to enforce these policies.

YAML
# Configure Istio VirtualService for LLM APIs
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: llm-virtual-service
spec:
  hosts:
  - "*"
  http:
  - route:
    - destination:
        host: llm-service
      weight: 100
      timeout: 30s
    telemetry:
      loggingLevel: debug
      overrideStats:
        quota-limits:
          maxRequestsPerMinute: "500"
Step 3: Implement Quota Enforcement Logic

Quotas can be enforced using Istio's built-in policy mechanisms or by integrating with a custom backend service. For simplicity, we'll use Istio for rule-based enforcement.

YAML
# Configure Istio DestinationRule to apply quotas
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: llm-destination-rule
spec:
  host: llm-service
  subsets:
  - name: quota-limited
    labels:
      app: llm-service
Step 4: Logging and Reporting

To log usage metrics, we can use Prometheus to capture data from Istio and then process it with Grafana or any other visualization tool. This allows us to generate detailed reports on token spend per team.

YAML
# Example Prometheus Rule for LLM API Metrics
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: llm-service-monitor
spec:
  selector:
    matchLabels:
      app: llm-service
  endpoints:
  - port: metrics
Step 5: Integration with Cost Management Tools

We can integrate our logging and reporting with cloud-native cost management tools like AWS Budgets, GCP Cost Explorer, or Azure Cost Management. These services provide insights into actual spending and help us compare it against our quotas.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, implementing a proxy/gateway using Istio provides the necessary capabilities to log, monitor, and enforce per-team token spend visibility for LLM APIs. This setup ensures that each team can stay within their budget and receive detailed reports on their usage.

#### Common Mistakes

Overlooking the need for custom metrics pipelines.
Failing to properly configure quotas and enforcement rules.
Not integrating effectively with cost management tools, leading to delayed or inaccurate reporting.

#### Follow-Ups

How would you handle cases where teams exceed their token limits?
What other security measures would you recommend in this setup?

**Code:**

```
Gateway Configuration Example

apiVersion: networking.istio.io/v1beta1 kind: Gateway metadata: name: llm-gateway spec: selector: istio: ingressgateway # use Istio default gateway implementation servers:

port: number: 80 name: http protocol: HTTP hosts:
"*"
CODE
Copy

```yaml
# VirtualService Configuration Example
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: llm-virtual-service
spec:
  hosts:
  - "*"
  http:
  - route:
    - destination:
        host: llm-service
      weight: 100
      timeout: 30s
    telemetry:
      loggingLevel: debug
      overrideStats:
        quota-limits:
          maxRequestsPerMinute: "500"
YAML
Copy
# DestinationRule Configuration Example
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: llm-destination-rule
spec:
  host: llm-service
  subsets:
  - name: quota-limited
    labels:
      app: llm-service
YAML
Copy
# ServiceMonitor Configuration Example
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: llm-service-monitor
spec:
  selector:
    matchLabels:
      app: llm-service
  endpoints:
  - port: metrics
```

**Example input / output:**

| Input (HTTP Request) | Output (Response) |
|----------------------|--------------------|
| `GET /api/v1/llm?team=DevTeam` | `{"message": "Request for DevTeam successful. Current usage: 5000 tokens."}` |

**Data:**

```json
{
  "teams": [
    {
      "name": "DevTeam",
      "tokenLimitPerMonth": 10000,
      "currentUsage": 5000
    },
    {
      "name": "ProdTeam",
      "tokenLimitPerMonth": 8000,
      "currentUsage": 3000
    }
  ]
}
```

