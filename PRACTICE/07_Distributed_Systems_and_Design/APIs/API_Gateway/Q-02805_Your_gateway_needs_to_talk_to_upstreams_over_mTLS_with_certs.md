---
type: "practice"
difficulty: "Deep"
---

# Your gateway needs to talk to upstreams over mTLS with certs rotated by a service mesh; how do you keep the gateway's client certs fresh, and what breaks during the rotation overlap window?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Responsibilities and Anti-patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Responsibilities_and_Anti-patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that an API Gateway maintains its client certificates (mTLS) up-to-date while dealing with certificate rotations orchestrated by a service mesh, we need to adopt a systematic approach involving both the gateway and the service mesh. This involves understanding how mTLS works, the lifecycle of certificates, and the potential risks during overlapping periods.

The core idea is that the API Gateway must stay in sync with the latest certificates from the service mesh without disrupting service availability. This solution requires careful planning, monitoring, and robust error handling mechanisms.

#### Triaging the Question
What is mTLS? - Mutual TLS (mTLS) involves both client and server verifying each other’s identities using X.509 certificates.
How does certificate rotation work in a service mesh? - Service meshes like Istio handle automatic certificate issuance, renewal, and revocation through its Citadel component.
What are the challenges during overlap periods? - The gateway must ensure it has the latest certificates without causing disruptions.

#### First-Principles Solution
Certificate Management: The API Gateway should use a dynamic approach to fetch the latest certificates from the service mesh.
Health Checks and Monitoring: Implement periodic checks to verify if the gateway’s certificates are valid and up-to-date.
Rolling Updates: Gradually update the gateway with new certificates while ensuring continuous availability.
Step-by-Step Solution

Dynamic Certificate Fetching:

The API Gateway should periodically check the service mesh for updated certificates using a REST API provided by the service mesh.
Use a library or SDK that supports mTLS to manage certificate fetching and re-validation.

Health Checks:

Implement health checks within the gateway to verify if it can still communicate securely with upstream services after fetching new certificates.
Set up alerts and logging for any failed health checks to quickly identify issues.

Rolling Updates:

Schedule a rolling update of the API Gateway pods while ensuring that at least one pod remains available during the transition.
Use Kubernetes labels or annotations to facilitate selective updates without affecting service availability.

#### Closing Summary

To keep an API Gateway's client certificates fresh and manage certificate rotations seamlessly, we need to implement dynamic certificate fetching, robust health checks, and rolling updates. This ensures minimal downtime and continuous secure communication between the gateway and upstream services.

#### Common Mistakes/Follow-Ups
Scheduling Cert Rotations Too Quickly: Rapid certificate rotations can overwhelm the API Gateway's ability to handle frequent updates.
Failing Health Checks: Not setting up proper monitoring for health checks may lead to unexpected service disruptions.
Downtime During Updates: Rolling updates should be carefully planned to minimize disruption, especially in high-traffic environments.

**Comparison:**

```
Step	Action
1	Fetch certificates from service mesh API.
2	Validate the fetched certificates.
3	Update gateway if necessary, monitoring health checks.
```

**Code:**

```python
# Example Python code snippet for fetching and validating certificates
import requests

def fetch_certs(service_mesh_api):
    response = requests.get(f"https://{service_mesh_api}/certificates")
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception("Failed to fetch certificates")

def validate_cert(cert_data):
    # Validate the certificate data
    pass

# Example function for rolling update
import kubernetes.client

def roll_update_gateway(namespace, pod_label_selector):
    api_instance = kubernetes.client.CoreV1Api()
    pods = api_instance.list_namespaced_pod(namespace, label_selector=pod_label_selector)
    for pod in pods.items:
        # Perform rolling update operations here
        pass
```

**Example input / output:**

```text
Input:
GET https://citadel.example.com/certificates

Output:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "certificates": [
    {
      "name": "gateway-cert-1234567890",
      "issued_to": "api-gateway",
      "issuer": "CA1234567890",
      "not_before": "2023-10-01T00:00:00Z",
      "not_after": "2024-10-01T00:00:00Z"
    }
  ]
}
```

**Data:**

```json
{
  "service_mesh_api": "https://citadel.example.com",
  "certificates": [
    {
      "name": "gateway-cert-1234567890",
      "issued_to": "api-gateway",
      "issuer": "CA1234567890",
      "not_before": "2023-10-01T00:00:00Z",
      "not_after": "2024-10-01T00:00:00Z"
    }
  ]
}
```

**Flow:**

1. Yes
2. Check for New Certificates
3. Are New Certificates Available?
4. Fetch and validate newcertificates
5. Update Gateway with newcerts if valid
6. Verify health of gateway afterupdate
7. Rolling Update Success
8. Rolling Update Failed

