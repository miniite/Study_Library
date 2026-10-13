---
type: "practice"
difficulty: "Deep"
---

# Your gateway routes to services in two Kubernetes clusters via a multi-cluster setup; how do you handle failover between clusters when one's control plane is unreachable but its data plane still serves?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Resilience to Dependencies](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Resilience_to_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a multi-cluster API Gateway setup, handling failover efficiently ensures high availability and robustness without compromising performance or user experience. The key idea here is to design a resilient gateway that can dynamically switch between clusters based on their current state—specifically, whether the control plane is reachable. If one cluster's control plane becomes unreachable but its data plane still serves requests, the API Gateway should seamlessly route traffic to the operational data plane.

This question probes your understanding of Kubernetes clustering, high availability strategies, and failover mechanisms. We'll start by establishing some assumptions:

You have two Kubernetes clusters: Cluster A (primary) and Cluster B (secondary).
The primary cluster has a working control plane but an unreachable or unhealthy service.
The secondary cluster's data plane can still handle incoming requests.
The API Gateway must be able to detect the state of each cluster.

To address this, we need to implement a robust health check mechanism for both clusters and a dynamic routing strategy that switches traffic from the primary to the secondary when necessary. Let’s break down the solution step-by-step:

Health Check Mechanism: We’ll set up an automated health check using Kubernetes CRDs (Custom Resource Definitions) or third-party tools like Prometheus, which can monitor the state of each cluster.
Dynamic Routing Strategy: The API Gateway will use this health check data to decide whether to route traffic to Cluster A or B. If Cluster A’s control plane is down but its services are still accessible via the data plane, the gateway should switch routing to Cluster B.
Failover Mechanism: Implement a failover mechanism that can detect and react to changes in cluster health.

Here's a detailed breakdown:

Health Check Mechanism
Prometheus Monitoring:
Deploy Prometheus to monitor the state of Kubernetes clusters.
Create alerts for control plane reachability and service status.
Custom Resource Definitions (CRDs):
Define CRDs that expose cluster health statuses.
Dynamic Routing Strategy
API Gateway Configuration:
Configure your API Gateway (e.g., NGINX, Istio) to dynamically update its routing rules based on the health status of each cluster.
Health Check Logic:
Implement a health check logic in your API Gateway that periodically queries Prometheus for current health statuses.
Update routing rules if a change is detected.
Failover Mechanism
Route Based on Health Status:
If Cluster A’s control plane becomes unreachable, the API Gateway should automatically switch to using services from Cluster B.
Graceful Degradation:
Ensure that any traffic still routed to an unhealthy cluster is handled gracefully (e.g., with fallback responses or retries).

#### Implementation
Example

Let's walk through a simple example using NGINX as our API Gateway and Prometheus for health checks.

Health Check Setup
BASH
# Install Prometheus in your monitoring environment
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.54.1-bugfix-2/example/prometheus-operator-crds.yaml
kubectl apply -f https://raw.githubusercontent.com/prometheus-community/prometheus-kubernetes/main/kube-state-metrics/deploy/kubernetes-apiserver.yml

# Create a Prometheus service monitor for Kubernetes clusters
cat <<EOF | kubectl apply -f -
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: cluster-monitoring
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: prometheus-operator
  endpoints:
    - port: web
      path: /api/v1/query?query=up{job=kube_apiserver}
EOF
NGINX Configuration Update
NGINX
# Nginx configuration for routing based on health status
http {
    upstream backend {
        server clusterA.example.com:80 fail_timeout=30s max_fails=1;
        server clusterB.example.com:80 backup;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
        }
    }
}
Failover Logic
If clusterA’s control plane is unreachable, NGINX will automatically switch to clusterB.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, a robust multi-cluster API Gateway setup requires a combination of health checks, dynamic routing strategies, and failover mechanisms. Common mistakes include:

Failing to implement timely health checks.
Not configuring the API Gateway to react dynamically to health status changes.
Overlooking graceful degradation for failed clusters.

#### Follow-up
questions might explore advanced topics such as zero-downtime deployments, multi-region configurations, or integrating with Kubernetes services like Service Mesh for enhanced reliability and observability.

**Comparison:**

```
Cluster	Control Plane Health	Data Plane Health
A	Unreachable	Operational
B	Reachable	Operational
```

**Example input / output:**

```text
# Sample Input/Output for Health Check and Routing

Input (Prometheus Query):
GET /api/v1/query?query=up{job=kube_apiserver}

Response:
{
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {},
        "value": ["2023-11-14T18:57:39Z", "1"]
      }
    ]
  }
}

# API Gateway Routing Configuration
http {
  upstream backend {
    server clusterA.example.com:80 fail_timeout=30s max_fails=1;
    server clusterB.example.com:80 backup;
  }

  server {
    listen 80;

    location / {
      proxy_pass http://backend;
      proxy_set_header Host $host;
    }
  }
}
```

**Data:**

```yaml
# Prometheus ServiceMonitor Example
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: cluster-monitoring
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: prometheus-operator
  endpoints:
    - port: web
      path: /api/v1/query?query=up{job=kube_apiserver}
```

**Flow:**

1. Control Plane Down
2. Data Plane Still Operational
3. Cluster A
4. Failover to Cluster B
5. Route Traffic via Cluster B

