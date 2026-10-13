---
type: "practice"
difficulty: "Expert"
---

# Your search platform's indexing pipeline is the bottleneck at 2M docs/sec across tenants. Redesign the write path — backpressure, prioritization, fair-sharing across tenants — so a bulk backfill from one tenant can't starve real-time updates for others.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Search Platform Scaling](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Platform_Scaling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To address the bottleneck in our indexing pipeline, we need to redesign it to handle 2M docs/sec efficiently while ensuring that backpressure is managed effectively and tenants are fairly shared across real-time updates. The key steps involve implementing a multi-tenant architecture with configurable backpressure mechanisms, prioritization policies, and fair-sharing strategies.

#### Triaging the Question

The core of this question revolves around optimizing the write path in an Elasticsearch-based search platform to ensure high throughput while maintaining fairness among tenants. Specifically, we need to:

Handle High Throughput: The system needs to process 2M documents per second.
Manage Backpressure: Prevent a bulk backfill from one tenant from starving other tenants of real-time updates.
Prioritize and Fairly Share Resources: Ensure that each tenant gets an equitable share of the indexing resources.

#### Assumptions
We are using Elasticsearch as our search platform, which handles both indexing and querying.
The system supports multiple tenants, each with its own set of documents.
Each document is indexed asynchronously to balance between high throughput and data consistency.
There are varying levels of urgency or importance among the different types of documents.
Step-by-Step Solution from First Principles

Multi-Tenant Architecture:

We will introduce a multi-tenant model where each tenant has its own namespace within Elasticsearch, ensuring isolation.

Backpressure Management:

Implement backpressure mechanisms to dynamically adjust indexing rates based on the current load. This can be done using rate limiting or queue-based approaches.

Prioritization Policies:

Define prioritization policies for different types of documents. For example, real-time updates could be given higher priority than bulk backfills.

Fair Sharing Strategy:

Use a fair-sharing mechanism to ensure that no single tenant monopolizes the indexing resources. This can be achieved using token bucket algorithms or other load balancing techniques.

Configuration and Monitoring:

Provide configuration options to tenants to adjust their indexing policies according to their needs.
Implement monitoring tools to track the performance of each tenant and make real-time adjustments as necessary.

#### Implementation
Details

Multi-Tenant Namespaces:

Tenant 1
tenant1_index
Tenant 2
tenant2_index
Shared Resources
Elasticsearch Cluster

Backpressure Mechanism:

Use a message queue (e.g., Kafka) to buffer incoming documents before indexing.
PYTHON
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')

def index_document(document):
    future = producer.send('index_topic', document)
    return future.get(timeout=10)

Prioritization Policies:

Assign priorities to different types of documents and use a priority queue.
PYTHON
from heapq import heappush, heappop

priority_queue = []

def add_document(document):
    heappush(priority_queue, (-document.priority, document))

def get_next_document():
    return heappop(priority_queue)[1]

Fair Sharing Strategy:

Use token bucket algorithm to limit the rate of indexing per tenant.
PYTHON
from threading import Lock

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.refill_rate = refill_rate
        self.tokens = 0
        self.lock = Lock()

    def acquire(self):
        with self.lock:
            if self.tokens > 0:
                self.tokens -= 1
            else:
                self.tokens += self.refill_rate * (time.time() - last_refill_time)

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
By implementing a multi-tenant architecture with backpressure mechanisms, prioritization policies, and fair-sharing strategies, we can ensure that the indexing pipeline handles 2M docs/sec efficiently while maintaining fairness among tenants.

#### Common Mistakes

Overlooking the importance of backpressure: A bulk backfill could easily overwhelm the system if not managed properly.
Ignoring tenant prioritization: Without proper prioritization, critical real-time updates might be delayed or starved.
Failing to monitor and adjust dynamically: Real-time monitoring and adjustments are crucial for maintaining optimal performance.

#### Follow-Ups

How do we handle different levels of urgency among documents?
Can you describe how the token bucket algorithm works in more detail?
What are some best practices for tuning the backpressure mechanisms?

**Comparison:**

| Tenant | Document Type | Priority Level |
|--------|---------------|----------------|
| A      | Real-time     | High           |
| B      | Historical    | Medium         |
| C      | Audit         | Low            |

**Code:**

```python
from kafka import KafkaProducer
from heapq import heappush, heappop

# Backpressure Mechanism using Kafka Producer
producer = KafkaProducer(bootstrap_servers='localhost:9092')

def index_document(document):
    future = producer.send('index_topic', document)
    return future.get(timeout=10)

# Prioritization Policies using Priority Queue
priority_queue = []

def add_document(document):
    heappush(priority_queue, (-document.priority, document))

def get_next_document():
    return heappop(priority_queue)[1]

# Fair Sharing Strategy using Token Bucket Algorithm
from threading import Lock

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.refill_rate = refill_rate
        self.tokens = 0
        self.lock = Lock()

    def acquire(self):
        with self.lock:
            if self.tokens > 0:
                self.tokens -= 1
            else:
                self.tokens += self.refill_rate * (time.time() - last_refill_time)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `add_document({"tenant": "customerA", "priority": 5, "document_id": "doc-1234"})` | Added document to priority queue with high priority. |
| `get_next_document()` | Returns the next highest-priority document for indexing: {"tenant": "customerA", "priority": 5, "document_id": "doc-1234"}. |

**Data:**

```json
{
  "tenant": "customerA",
  "priority": 5,
  "document_id": "doc-1234"
}
```

**Flow:**

1. Document Arrival
2. Buffer Queue
3. Token Bucket Check
4. Indexing Process
5. Tenant Namespace
6. Backpressure Mechanism
7. Prioritization Policy
8. Fair Sharing Strategy

