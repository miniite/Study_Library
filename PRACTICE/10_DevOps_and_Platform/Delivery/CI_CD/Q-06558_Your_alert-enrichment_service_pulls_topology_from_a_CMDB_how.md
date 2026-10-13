---
type: "practice"
difficulty: "Intermediate"
---

# Your alert-enrichment service pulls topology from a CMDB; how would you deploy a change to the enrichment logic without a stale CMDB snapshot in CI giving you false confidence?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Expand and Contract Migrations](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Expand_and_Contract_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of deploying a change to the alert-enrichment service's logic without relying on a potentially outdated CMDB, we need to ensure that our deployment process is robust and can handle changes in real-time or near-real-time. This involves designing a CI/CD pipeline that minimizes the risk of false confidence due to stale data.

The key idea here is to decouple the deployment of the enrichment logic from the state of the CMDB, ensuring that any updates to the service are validated against the latest and most accurate topology information. We can achieve this by integrating a real-time or near-real-time update mechanism into our pipeline.

#### Triaging the Question

The core question revolves around deploying changes in a CI/CD context while maintaining data consistency despite potential delays in CMDB updates. The interviewer is probing your understanding of how to manage dependencies, decouple systems, and handle asynchronous data flows effectively.

We assume:

The CMDB provides topology information that is critical for the alert-enrichment service.
There may be a delay between when changes are made to the CMDB and when they become visible to our services.
We need to ensure that any new deployment of the enrichment logic is validated against the latest data.

#### First Principles Solution

Real-Time Data Ingestion: Implement real-time or near-real-time updates from the CMDB into a local cache within the alert-enrichment service. This can be done using an event-driven architecture where changes to the CMDB trigger updates in our service.

Decoupled Deployment: Deploy changes to the enrichment logic independently of when the CMDB is updated. Ensure that any new deployment includes a validation step against the latest topology data stored locally.

Validation Mechanism: Introduce a mechanism within the enrichment logic to validate configurations or dependencies based on the local cache of the latest CMDB information, even if this information might lag slightly behind the actual state in the CMDB.

Step-by-Step Solution

Set Up Real-Time Data Ingestion:

Use a streaming service (e.g., Kafka, NATS) to ingest events from the CMDB.
Store these events in a local cache or database within the alert-enrichment service.

Deployment Process:

Integrate the deployment process with a CI/CD pipeline that triggers on code changes or manual deployments.
During the deployment, validate the new configuration against the latest topology data stored locally.

Validation Step:

Implement a validation step in your enrichment logic to check for consistency between the local cache and any external dependencies.
Use this step as part of the initial startup sequence of the service or during periodic checks to ensure data integrity.

#### Closing Summary

By implementing real-time updates from the CMDB into our service, we can decouple the deployment process from the state of the CMDB. This ensures that any changes to the alert-enrichment logic are validated against the latest topology information, thereby minimizing false confidence due to stale data.

#### Common Mistakes
and Follow-Ups:

Failing to properly handle real-time updates and relying solely on periodic checks.
Not integrating a robust validation mechanism into the deployment process.
Overlooking the need for a decoupled design that can handle asynchronous data flows effectively.

**Comparison:**

```
Event Type	Node ID	Attributes
add_node	12345	{hostname: "node-01", role: "web"}
remove_edge	-	source: 12345, dest: 67891
```

**Code:**

```python
# Example of a simple validation function in Python
def validate_topo(topology_data):
    # Logic to check if local cache matches external dependencies
    pass

# Sample CMDB event handling
class CmdbEventHandler:
    def __init__(self, service_cache):
        self.service_cache = service_cache

    def handle_cmdb_event(self, event):
        self.service_cache.update(event)

        # Run validation after updating the cache
        validate_topo(self.service_cache.topology_data)

# Example of deployment trigger in CI/CD pipeline
def deploy_enrichment_logic():
    # Trigger validation before deploying new logic
    validate_topo(get_latest_topology())

    # Proceed with deployment
    update_service()
```

**Example input / output:**

```text
Input (CMDB event):
cmdb_events = [
  {"type": "add_node", "node_id": 12345, "attributes": { "hostname": "node-01", "role": "web" }},
  {"type": "remove_edge", "edge_id": 67890, "source_node_id": 12345, "destination_node_id": 67891}
]

Output (Service cache update and validation):
Updated service_cache.topology_data:
nodes: [ {id: 12345, attributes: {hostname: "node-01", role: "web"}} ]
edges: []

Validation result: Successful
```

**Data:**

```yaml
cmdb_events:
  - type: add_node
    node_id: 12345
    attributes: {hostname: "node-01", role: "web"}
  - type: remove_edge
    edge_id: 67890
    source_node_id: 12345
    destination_node_id: 67891

service_cache:
  topology_data:
    nodes:
      - id: 12345
        attributes: {hostname: "node-01", role: "web"}
    edges:
      - source: 12345
        destination: 67891
```

**Flow:**

1. Yes
2. No
3. CMDB Event
4. Is Real-Time?
5. Update Service Cache
6. Periodic Check (Optional)
7. Run Validation Against Cache
8. Perform Deployment
9. Deployment Success
10. Validation Failed
11. Proceed with Deployment
12. Handle Error/Retry Logic

