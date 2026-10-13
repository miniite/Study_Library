---
type: "practice"
difficulty: "Intermediate"
---

# You correlate alerts by a topology graph fetched from a CMDB on the hot path, and a CMDB blip spiked correlation latency and split incidents. Explain how you'd cache the topology locally with a TTL and safe fallback so the CMDB is never in the critical correlation path.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gossip and Deduplication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Gossip_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When correlating alerts using a topology graph from a Configuration Management Database (CMDB), caching the topology locally can significantly improve performance and reduce latency. For AIOps at intermediate experience level, we'll design a solution that includes local caching with a Time-to-Live (TTL) to minimize dependencies on the CMDB, while ensuring safe fallback mechanisms when necessary.

#### Triaging the Question

The core question revolves around optimizing alert correlation by reducing the reliance on real-time data from the CMDB. The CMDB is prone to latency spikes, which can negatively impact critical incident handling processes. To mitigate this, we need a robust caching strategy that ensures availability and consistency of topology data.

#### Assumptions
Topology Graph: We assume the topology graph is stored in a structured format such as JSON or a graph database.
TTL Mechanism: The cache will have an expiration time to ensure it doesn’t become stale, but also needs a safe fallback mechanism when the local cache expires.
Incident Handling: The system should handle both normal and abnormal conditions gracefully without causing critical failures.
Step-by-Step Solution
Fetch Topology Graph from CMDB:
On initial startup or during periodic updates, fetch the topology graph from the CMDB.
Local Cache with TTL:
Store the fetched topology in a local cache (e.g., Redis) with an appropriate Time-to-Live (TTL).
Fallback Mechanism:
Implement a fallback mechanism to query the CMDB directly when the local cache expires or is unavailable.
Monitoring and Alerts:
Monitor the health of the local cache and CMDB, and set up alerts for critical issues like cache expiry or CMDB unavailability.
Detailed Implementation
Step 1: Fetch Topology Graph

When the system starts, it should fetch the topology graph from the CMDB. This can be done using a standard API call:

PYTHON
def fetch_topology_graph():
    response = requests.get('https://cmdb.example.com/topology')
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception("Failed to fetch topology graph from CMDB")
Step 2: Local Cache with TTL

Use a distributed cache like Redis to store the fetched topology. Here’s an example using Python and Redis-py library:

PYTHON
import redis

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def cache_topology(topology):
    key = 'topology'
    expiration_time = 3600  # Cache expires in 1 hour
    redis_client.set(key, topology)
    redis_client.expire(key, expiration_time)
Step 3: Fallback Mechanism

Implement a function to check if the local cache is still valid. If not, fallback to querying the CMDB:

PYTHON
def get_topology_graph():
    key = 'topology'
    topology = redis_client.get(key)
    if topology is None or time.time() - int(redis_client.ttl(key)) > expiration_time:
        try:
            topology = fetch_topology_graph()
            cache_topology(topology)
        except Exception as e:
            # Log the error and return a default topology
            logger.error(f"Failed to fetch topology from CMDB: {e}")
            return get_default_topology()  # Implement this function to provide a fallback
    return topology
Step 4: Monitoring and Alerts

Set up monitoring for both the local cache and CMDB. Use tools like Prometheus or Grafana to monitor cache expiration times and CMDB health:

PLAINTEXT
- Alert when cache expires before its TTL
- Monitor CMDB response time and availability

#### Closing Summary
and Common Mistakes
Key Points:
Cache Strategy: Utilize a local cache with a TTL.
Fallback Mechanism: Ensure safe fallback to the CMDB if needed.
Monitoring: Set up alerts for critical issues.

#### Common Mistakes
Ignoring Cache Expiry: Forgetting to set or check the cache expiration time.
Failing to Implement Fallback: Not having a robust mechanism to query the CMDB when necessary.
Poor Monitoring: Overlooking the importance of monitoring and alerting on cache and database health.

**Code:**

```python
import requests
from datetime import datetime, timedelta

def fetch_topology_graph():
    response = requests.get('https://cmdb.example.com/topology')
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception("Failed to fetch topology graph from CMDB")

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def cache_topology(topology):
    key = 'topology'
    expiration_time = 3600  # Cache expires in 1 hour
    redis_client.set(key, topology)
    redis_client.expire(key, expiration_time)

def get_topology_graph():
    key = 'topology'
    topology = redis_client.get(key)
    if topology is None or time.time() - int(redis_client.ttl(key)) > expiration_time:
        try:
            topology = fetch_topology_graph()
            cache_topology(topology)
        except Exception as e:
            logger.error(f"Failed to fetch topology from CMDB: {e}")
            return get_default_topology()  # Implement this function
    return topology

def get_default_topology():
    # Default logic or configuration for fallback
    pass
```

**Example input / output:**

```text
Input:
- Initial call to `get_topology_graph()`

Output:
- Topology graph fetched from local cache if available, or from CMDB if not.

Fallback Response (if cache expires):
- Fallback logic in place and default topology provided.
```

**Data:**

```json
{
  "nodes": [
    {"id": "Node1", "name": "Switch A"},
    {"id": "Node2", "name": "Server B"}
  ],
  "edges": [
    {"source": "Node1", "target": "Node2"}
  ]
}
```

