---
type: "practice"
difficulty: "Deep"
---

# A serving change swapped from a managed API to self-hosted vLLM; how would you make CI verify functional and cost parity before cutting production traffic over?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Validating Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a transition from a managed API to a self-hosted vLLM in a serving environment maintains both functionality and cost parity, we need to rigorously validate multiple aspects of the system using comprehensive CI/CD pipelines. This involves several steps, including functional testing, performance analysis, resource utilization monitoring, and cost comparison. We will break down each component from first principles.

#### Triage

The core question is how to design a CI/CD pipeline that can thoroughly test the self-hosted vLLM before fully switching it in production. The key aspects are:

Functional Testing: Ensuring the new service behaves identically to the managed API.
Performance Analysis: Comparing the response times and throughput of both services.
Resource Utilization Monitoring: Checking that the self-hosted vLLM does not exceed its resource limits.
Cost Comparison: Verifying that the transition does not increase operational costs significantly.

#### Assumptions
We assume the managed API has a well-defined set of endpoints and response formats.
The self-hosted vLLM is available as an API or can be integrated into existing services.
Both systems are observable through standard logging and monitoring tools.
There is a budget for increased costs during the transition period.

#### First-Principles Solution
Setup Testing Environments: Create test environments that replicate production conditions, including database schemas, APIs, and resource constraints.
Functional Tests:
Write unit tests to validate individual components of the vLLM.
Develop integration tests to ensure that vLLM endpoints return the same responses as the managed API.
Performance Tests:
Use tools like Apache JMeter or LoadRunner to generate load and compare response times and throughput between the two services.
Resource Utilization Monitoring:
Monitor CPU, memory, network usage, and other key metrics using Prometheus or similar monitoring systems.
Set up alerting for potential resource constraints that could lead to cost overruns.
Cost Comparison:
Use cloud provider APIs (like AWS Cost Explorer) to track costs associated with the managed API and vLLM service.
Ensure that any increased usage during testing is within budget.

#### Implementation
Steps

Environment Setup:

SH
# Example Docker Compose file for setting up test environments
version: '3'
services:
  db:
    image: postgres:latest
    environment:
      POSTGRES_PASSWORD: example
  api-mock:
    image: managed-api-service
  vllm-svc:
    build: ./vllm-service
    environment:
      VLLM_API_URL: http://api-mock:8000/api/v1

Functional Testing:

Use a testing framework like pytest for unit and integration tests.
PYTHON
# Example test case using pytest
import requests

def test_vllm_endpoint():
    response = requests.get('http://vllm-svc:8000/api/v1/endpoint')
    assert response.status_code == 200
    assert response.json() == expected_response

Performance Testing:

Use JMeter to simulate load and compare performance.
SH
# Example JMeter command
jmeter -t test.jmx -l result.jtl

Monitoring Setup:

Configure Prometheus and Grafana to collect and visualize resource usage metrics.
YAML
# Example Prometheus configuration
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'vllm-service'
    static_configs:
      - targets: ['vllm-svc:9090']

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By systematically validating both the functional and performance aspects, we can ensure that a transition from managed to self-hosted vLLM maintains parity in production. This approach helps avoid unexpected issues and cost overruns.

#### Common Mistakes
Skipping thorough testing, leading to integration bugs or performance degradation.
Failing to monitor resource utilization effectively, resulting in service outages due to resource exhaustion.
Not properly accounting for cost implications during the transition period.

For follow-ups, consider discussing how to integrate machine learning-specific tests (e.g., accuracy of predictions) and how to handle edge cases where API responses differ slightly. Also, explore the impact on cold start times and latency in real-world scenarios.

**Comparison:**

```
Metric	Expected (Managed API)	Observed (vLLM)
Response Time	200 ms	195 ms
Throughput	500 req/s	480 req/s
```

**Code:**

```python
# Example test case using pytest
import requests

def test_vllm_endpoint():
    response = requests.get('http://vllm-svc:8000/api/v1/endpoint')
    assert response.status_code == 200
    assert response.json() == expected_response
```

**Example input / output:**

```text
# Example JMeter command to simulate load
jmeter -t test.jmx -l result.jtl
```

**Data:**

```yaml
# Example Docker Compose file for test environments
version: '3'
services:
  db:
    image: postgres:latest
    environment:
      POSTGRES_PASSWORD: example
  api-mock:
    image: managed-api-service
  vllm-svc:
    build: ./vllm-service
    environment:
      VLLM_API_URL: http://api-mock:8000/api/v1
```

**Flow:**

1. Environment Setup
2. Functional Tests
3. Performance Tests
4. Resource Monitoring
5. Cost Comparison

