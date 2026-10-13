---
type: "practice"
difficulty: "Expert"
---

# Your gateway must migrate from IP-based trust to workload-identity (SPIFFE/mTLS) for all upstream calls across hundreds of services; sequence this multi-year migration so there's never a window where valid workloads are rejected or unauthenticated ones slip through.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Responsibilities and Anti-patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Responsibilities_and_Anti-patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate from IP-based trust to workload-identity (SPIFFE/mTLS) for all upstream calls across hundreds of services, we need to ensure a smooth and secure transition without any downtime or misauthentication. This multi-year migration plan must be carefully orchestrated to maintain service availability and security throughout the process.

The key idea is to gradually shift each service from IP-based authentication to SPIFFE/mTLS, ensuring that during any given moment, at least one form of authentication remains robust and reliable.

First, we will set up a secure environment for SPIFFE/mTLS by integrating it with our existing identity provider. Then, we'll create a phased migration plan that includes setting up trust anchors, configuring service identities, and updating each service to validate both IP-based and mTLS tokens until the final transition is complete.

**Comparison:**

```
Phase	Authentication Methods	Service States	Notes
1	Both IP and mTLS	All services	Ensuring smooth transition
2	Only IP	Some services	Partially migrated
3	Only mTLS	All services	Complete migration completed
```

**Code:**

```python
# Example of SPIFFE configuration in Python (simplified)
import spiffe.config

def setup_spiffe_trust_anchors(anchors_file):
    config = spiffe.config.load_config_from_file(anchors_file)
    spiffe.set_trust_anchors(config.anchors)

setup_spiffe_truste_anchors('path/to/trust/anchors.json')
```

**Example input / output:**

```

To illustrate the phased migration, consider a simple example where ServiceA is migrating from IP-based to SPIFFE/mTLS. Initially, both forms of authentication are accepted.

Initial State: Both IP and mTLS tokens accepted.
Phase 1 (Partial Migration): Only IP tokens are accepted by ServiceA.
Phase 2 (Migration in Progress): mTLS tokens are now enforced but IP tokens continue to work as a fallback.
Final State: Only mTLS tokens are accepted, and the migration of other services is complete.
PLAINTEXT
Copy
+-----------------+------------+
| Token Type      | Acceptance |
+-----------------+------------+
| IP              | Yes        |
+-----------------+------------+
| mTLS            | Yes        |
+-----------------+------------+
```

**Data:**

```json
{
  "trust_anchors": [
    { "uri": "spiffe://example.org/root-ca1", "priority": 1 },
    { "uri": "spiffe://example.org/root-ca2", "priority": 2 }
  ]
}
```

**Flow:**

1. Initial State
2. Phase 1 - Only IP accepted
3. Phase 2 - mTLS and IPaccepted
4. Final State - Only mTLSaccepted

