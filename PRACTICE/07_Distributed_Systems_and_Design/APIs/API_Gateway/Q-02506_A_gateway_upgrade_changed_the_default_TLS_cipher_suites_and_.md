---
type: "practice"
difficulty: "Deep"
---

# A gateway upgrade changed the default TLS cipher suites and old IoT clients can no longer connect; how do you detect the affected client population and roll a compatibility policy without weakening security for modern clients?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Responsibilities and Anti-patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Responsibilities_and_Anti-patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To address this challenge, we need to identify which IoT clients are impacted by the new TLS cipher suites and ensure that any required changes can be rolled out gradually without compromising overall security.

#### Triaging the Question

The key here is to detect which old IoT clients cannot connect due to the updated TLS settings. We also need a strategy for rolling out compatibility without reducing the security posture of modern clients, who are already using more secure cipher suites.

#### Assumptions
TLS Update: The gateway now uses a new set of TLS cipher suites that might be incompatible with older IoT clients.
Client Diversity: IoT clients vary in their cryptographic support and versions.
Security Constraints: We must maintain or enhance the security posture for modern clients while addressing compatibility issues.

#### First-Principles Solution
1. Identify Affected Clients
Logs Analysis: Analyze logs from the gateway to identify failed TLS handshakes from IoT clients.
Client Identification: Ensure each client sends a unique identifier in its connection attempts, which can be logged and correlated.
SQL
SELECT client_id, COUNT(*) as failed_handshake_count
FROM gateway_logs
WHERE status = 'failed_tls'
GROUP BY client_id;
2. Impact Assessment
Client Compatibility Matrix: Create a matrix mapping different IoT client types to their supported TLS cipher suites.
YAML
clients:
  - device_id: "12345"
    tls_versions: ["TLSv1", "TLSv1.1"]
    cipher_suites: []
  - device_id: "67890"
    tls_versions: ["TLSv1.2", "TLSv1.3"]
    cipher_suites: ["ECDHE_RSA_AES_128_GCM_SHA256", "DHE_RSA_AES_256_GCM_SHA384"]
3. Rollout Strategy
Phased Rollout: Gradually introduce the new compatibility policy to avoid overwhelming the system.
Whitelisting and Blacklisting: Implement a mechanism where modern clients are automatically whitelisted for enhanced security, while older clients require explicit handling.
PYTHON
def update_client_policy(client_id):
    client_data = get_client_details(client_id)
    if is_compatible(client_data):
        whitelist_client(client_id)
    else:
        handle_compatibility(client_id)
4. Security and Compliance
Security Audit: Conduct a thorough audit of the new policy to ensure it does not introduce any vulnerabilities.
Monitoring: Continuously monitor the gateway for any security breaches or issues related to the updated cipher suites.

#### Closing Summary

By identifying affected clients through log analysis, creating a client compatibility matrix, and implementing a phased rollout strategy, we can address the issue of older IoT clients without compromising the security posture for modern ones. Regular monitoring and audits will ensure that the solution remains robust and secure.

#### Common Mistakes/Follow-Ups
Failing to Whitelist Modern Clients: Ensuring modern clients are automatically whitelisted could avoid potential vulnerabilities.
Overly Aggressive Rollout: A gradual approach prevents overwhelming system resources and ensures a smooth transition.
Security Breaches: Regular audits and monitoring are crucial to detect any issues early.

**Comparison:**

```
Client ID	TLS Versions	Cipher Suites
12345	TLSv1, TLSv1.1	[]
67890	TLSv1.2, TLSv1.3	ECDHE_RSA_AES_128_GCM_SHA256, DHE_RSA_AES_256_GCM_SHA384
```

**Code:**

```python
def get_client_details(client_id):
    # Simulate client data retrieval
    return {
        'client_id': client_id,
        'tls_versions': ['TLSv1.2'],
        'cipher_suites': ['ECDHE_RSA_AES_128_GCM_SHA256']
    }

def is_compatible(client_data):
    supported_ciphers = ["ECDHE_RSA_AES_128_GCM_SHA256", "DHE_RSA_AES_256_GCM_SHA384"]
    return all(c in client_data['cipher_suites'] for c in supported_ciphers)

def whitelist_client(client_id):
    # Simulate client whitelisting
    print(f"Whitelisted client {client_id}")

def handle_compatibility(client_id):
    # Simulate compatibility handling
    print(f"Handling compatibility for client {client_id}")
```

**Example input / output:**

```text
Input:
- Client ID: 12345

Output:
Handling compatibility for client 12345
```

**Data:**

```yaml
clients:
  - device_id: "12345"
    tls_versions: ["TLSv1", "TLSv1.1"]
    cipher_suites: []
  - device_id: "67890"
    tls_versions: ["TLSv1.2", "TLSv1.3"]
    cipher_suites: ["ECDHE_RSA_AES_128_GCM_SHA256", "DHE_RSA_AES_256_GCM_SHA384"]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Client Connection
6. Is Client ID Known?
7. Retrieve Client Details
8. Is Client Compatible with NewPolicy?
9. Handle Compatibility
10. Whitelist Client
11. Log Failed Handshake andIdentify Client

