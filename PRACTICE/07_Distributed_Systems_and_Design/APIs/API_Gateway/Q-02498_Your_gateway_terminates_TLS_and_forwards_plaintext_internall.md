---
type: "practice"
difficulty: "Deep"
---

# Your gateway terminates TLS and forwards plaintext internally, but a compliance audit requires encryption in transit end to end; how do you add mTLS to all upstreams without a latency or ops explosion?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Security and Data Masking](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Security_and_Data_Masking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of implementing mutual TLS (mTLS) across all upstream services while maintaining performance and operational efficiency, we need to rethink our API Gateway architecture. The goal is to terminate TLS at the gateway but ensure that data remains encrypted between the gateway and internal services. Here’s a step-by-step approach:

#### Overview
We will introduce mTLS by modifying the existing API Gateway to handle mutual authentication with all upstream services while maintaining performance.

#### Triage
The question asks how to implement mTLS for end-to-end encryption without causing significant latency or operational overhead. This involves understanding current architecture, TLS termination points, and potential impact on service communication.

#### First-Principles Solution

Identify Upstream Services: List all upstream services that need to communicate with the API Gateway.
Configure mTLS Certificates: Obtain and install mTLS certificates for both the gateway and each upstream service.
Update Gateway Configuration: Modify the API Gateway configuration to enable mTLS on the communication channel between itself and the upstream services.
Test and Validate: Perform thorough testing to ensure that all services can communicate securely without impacting performance.

Detailed Steps:

#### Assumptions

The current gateway is already terminating TLS at the edge, which means it has the necessary infrastructure (e.g., Load Balancer or a custom solution).
Upstream services are accessible via well-defined endpoints.
There is an existing certificate management and distribution process in place.

Step-by-Step Solution:

Gather mTLS Requirements: Collaborate with the security team to gather necessary certificates for each upstream service. Ensure that these certificates are valid, not expired, and properly configured for mutual authentication.
Gateway Configuration Update: Modify the gateway configuration to enable client-side verification (peer verification) when communicating with upstream services. This typically involves updating TLS settings in the gateway’s configuration files or management console.
Service Registration and mTLS Setup:
Register each upstream service with a certificate authority if they do not already have one.
Ensure that these services are configured to present their client certificates when communicating with the API Gateway.
Testing: Perform load testing and functional testing to ensure that all services can communicate securely without performance degradation.
Monitoring and Logging: Implement comprehensive monitoring and logging for both the gateway and upstream services to detect any issues related to mTLS.

#### Closing Summary

By implementing mTLS, we can achieve end-to-end encryption between the gateway and internal services, enhancing security while maintaining performance. This involves configuring certificates, updating gateway settings, and conducting thorough testing.

#### Common Mistakes/Follow-Ups

Mistake 1: Failing to perform comprehensive testing before full deployment, leading to operational issues post-launch.

#### Follow-Up
Ensure detailed logging and monitoring are in place for early detection of any authentication or connection failures.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Gateway Config Update] --> B[Modify TLS Settings];
    B --> C[Service Registration];
    C --> D[Generate Certificates];
    D --> E[Configure Upstream Services];
    E --> F[Test and Validate];
```

**Example input / output:**

```text
Gateway Configuration Update:
Initial:
- TLS termination enabled on gateway

New Configuration:
- Client-side verification (peer verification) enabled for upstream communication

Upstream Service Registration and Setup:
Initial:
- No mTLS setup

After Changes:
- Upstream services registered with CA and configured to present client certificates
```

**Data:**

```yaml
api-gateway:
  tls-config:
    client-auth: required
    certificate: /path/to/gateway/cert.pem
    key: /path/to/gateway/key.pem
upstream-service1:
  tls-config:
    ca-cert: /path/to/ca/cert.pem
    cert: /path/to/upstream/cert.pem
    key: /path/to/upstream/key.pem
```

