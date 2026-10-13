---
type: "practice"
difficulty: "Intermediate"
---

# You need to expose the gateway's own metrics for an SLO dashboard; which RED/USE signals from Envoy or Kong actually matter for on-call, and how do you avoid a Prometheus scrape that itself overloads the proxy at high route cardinality?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Cost Metering and Budgets](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/Gateway_Cost_Metering_and_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To expose the gateway's own metrics in an SLO (Service Level Objective) dashboard, it is crucial to understand which signals from Envoy or Kong are critical for on-call engineers and how to prevent a Prometheus scrape from overwhelming the proxy at high route cardinality. In this scenario, we will focus on Envoy as our API Gateway implementation.

TL;DR

The key metrics to monitor include envoy_cluster_health, envoy_request_total, and envoy_upstream_cx_active. To avoid overloading the proxy, we can use a scrape_interval with Prometheus and implement rate limiting on the scrape endpoint in Envoy. Additionally, leveraging Prometheus's max_scrapes_per_minute setting can help manage high route cardinality.

#### Triage

The question asks us to identify relevant metrics from Envoy for on-call engineers and ensure that scraping these metrics does not overload the proxy. The goal is to balance visibility with performance optimization.

#### Assumptions
We are using Envoy as our API Gateway.
Prometheus is used for metric collection.
High route cardinality poses a risk of overloading the gateway.

#### Solution

To address the issue, we need to carefully select and implement metrics that provide actionable insights without compromising performance.

Step 1: Identify Critical Metrics

The most important metrics from Envoy are:

envoy_cluster_health: Provides health checks for individual clusters.
envoy_request_total: Tracks total requests passed through the gateway.
envoy_upstream_cx_active: Indicates active upstream connections, which helps in understanding load distribution.
Step 2: Configure Prometheus Scraping

To avoid overloading Envoy with a frequent scrape interval, we can use a combination of scrape_interval, max_scrapes_per_minute, and rate limiting on the scrape endpoint.

Set a Reasonable Scrape Interval:

A scrape interval of 5-10 seconds is often sufficient for most environments.
YAML
scrape_interval: 5s

Configure max_scrapes_per_minute:

This setting limits the number of scrapes from a single target in a given minute, preventing excessive load.
YAML
max_scrapes_per_minute: 10

Implement Rate Limiting on Scrape Endpoint (Envoy):

Use envoy.route to limit requests to the Prometheus scrape endpoint based on IP address or other criteria.
YAML
route:
  match:
    prefix: /metrics
  route:
    cluster: prometheus_cluster
  rate_limit:
    max_tokens_per_interval: 100
    interval_ms: 5000
Step 3: Monitor and Optimize
Monitor Gateway Health:
Continuously monitor the envoy_cluster_health metric to ensure all services are healthy.
Adjust Metrics Based on Load:
If high route cardinality causes issues, adjust the max_scrapes_per_minute setting dynamically based on observed load.

#### Closing Summary

By carefully selecting relevant metrics and implementing rate limiting on the scrape endpoint, we can expose essential gateway metrics to an SLO dashboard without overloading the Envoy proxy. This approach ensures both visibility and performance optimization.

#### Common Mistakes
and Follow-Ups
Over-scraping: Failing to set a reasonable scrape_interval or max_scrapes_per_minute can lead to excessive load.
Inadequate Metrics Selection: Not choosing the right metrics can result in missed critical insights or unnecessary data collection.
Dynamic Rate Limiting: For environments with varying load, dynamically adjusting rate limits based on observed traffic is crucial.

Do you have any follow-up questions on this setup?

**Code:**

```
Prometheus configuration snippet

scrape_interval: 5s max_scrapes_per_minute: 10

CODE
Copy

```yaml
# Envoy configuration snippet for rate limiting scrape endpoint

route:
  match:
    prefix: /metrics
  route:
    cluster: prometheus_cluster
  rate_limit:
    max_tokens_per_interval: 100
    interval_ms: 5000
```

**Example input / output:**

```
Example input to Envoy configuration

route: match: prefix: /metrics route: cluster: prometheus_cluster rate_limit: max_tokens_per_interval: 100 interval_ms: 5000

CODE
Copy

```plaintext
# Prometheus scrape configuration example

scrape_interval: 5s
max_scrapes_per_minute: 10
```

**Flow:**

1. Envoy Route Configuration
2. Rate Limit Tokens
3. Prometheus Scraping
4. Gateway Health Monitoring
5. Dynamic Adjustment of RateLimits

