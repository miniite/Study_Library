---
type: "practice"
difficulty: "Deep"
---

# How do you decide what belongs in the operational plane (OLTP/event log) vs the analytical plane (lake/warehouse), and how do you keep the coupling between them from becoming a distributed monolith?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Warehouse as System of Record](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide where data should reside in an operational plane (OLTP/event log) versus an analytical plane (lake/warehouse), it’s crucial to consider the requirements of each plane, their respective storage characteristics, and the trade-offs involved in maintaining a decoupled architecture. The key is to minimize coupling between these planes while ensuring that they can effectively interact to support business needs.

#### Triaging the Question

The interviewer is probing your understanding of data architecture principles, specifically focusing on how to balance operational and analytical workloads. They are also interested in your ability to manage complexity and maintain a clean separation of concerns, which is essential for scalable systems.

#### Assumptions
Operational Plane (OLTP/Event Log): Typically used for real-time transactional processing, where data must be available quickly.
Analytical Plane (Lake/Warehouse): Designed to handle large volumes of historical and processed data, often with complex queries and transformations.

#### First-Principles Solution
Data Purpose: Analyze the purpose of each dataset. Operational data is often transactional and requires low-latency access for immediate decisions, whereas analytical data is used for long-term trend analysis, reporting, or machine learning.
Latency Requirements: Consider the need for real-time updates vs batch processing. OLTP systems typically require sub-second response times, while analytics can tolerate longer latencies.
Data Volume and Type: Operational data often has high write frequency but low read frequency, whereas analytical data might have lower write rates but frequent reads. The type of data (structured, semi-structured) also influences storage choices.
Storage Costs: OLTP databases are optimized for transactional workloads with higher costs associated with frequent writes and strong consistency guarantees. Analytical systems like data lakes or warehouses can leverage cheaper storage options due to their batch processing nature.
Security and Compliance: Ensure that sensitive data is appropriately stored in secure environments, which might differ between operational and analytical planes.
Keeping Coupling Minimal

To maintain a decoupled architecture:

Event Sourcing: Use event sourcing for OLTP systems to capture all state changes as events, allowing these events to be written to both the operational database and an event log or lake.
Data Pipelines: Implement data pipelines that process events from the operational system into the analytical system, ensuring that updates in one plane do not directly affect the other.
APIs and Interfaces: Use well-defined APIs to interact between planes, minimizing direct dependencies. RESTful services, GraphQL, or message queues can serve as intermediaries.

#### Closing Summary

By carefully considering the purpose, latency requirements, volume of data, and security needs, you can effectively decide where data should reside in an operational versus analytical plane. Keeping coupling minimal ensures that changes in one plane do not inadvertently affect the other, preventing a distributed monolith from forming. This approach helps maintain system scalability and robustness.

#### Common Mistakes/Follow-Ups
Overlapping Responsibilities: Forgetting to clearly define roles between planes can lead to redundant systems or underutilized resources.
Inadequate Data Quality: Failing to ensure data integrity in both planes can result in inconsistent analytics.
Performance Bottlenecks: Not properly sizing and tuning your architectures for specific workloads can lead to performance issues.

Would you like me to walk through a sample scenario or discuss any specific technologies related to this concept?

**Comparison:**

```
Data Plane	Purpose	Latency	Volume	Storage Costs	Examples
Operational	Real-time transactions	< 1 sec	High writes, low reads	Expensive	RDBMS, Event Sourcing
Analytical	Historical data analysis	> 10 sec	Low writes, high reads	Cheaper	Data Lakes, Warehouses
```

**Code:**

```python
# Example of event sourcing in an OLTP system (simplified)
class Order:
    def __init__(self, order_id):
        self.order_id = order_id

class OrderEventBus:
    def publish_event(self, event):
        # Publish events to both operational and analytical systems
        pass

order_bus = OrderEventBus()
order = Order(order_id=123)
order_bus.publish_event(OrderPlaced(event_data={'order_id': 123}))
```

**Example input / output:**

```text
Input:
- Order placed with ID 123

Output:
- Order event recorded in both operational and analytical systems.
- Order status updated in OLTP database.
- Event written to event log for potential future analysis.
```

**Data:**

```json
{
    "orders": [
        {"order_id": 123, "status": "placed"},
        {"order_id": 456, "status": "shipped"}
    ],
    "events": [
        {"event_type": "OrderPlaced", "order_id": 123},
        {"event_type": "OrderShipped", "order_id": 456}
    ]
}
```

**Flow:**

1. Operational Plane
2. Event Sourcing
3. Write Events to Operational DB
4. Write Events to Event Log
5. Update OLTP Status
6. Store for Analysis
7. Query/Analyze Data

