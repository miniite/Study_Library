---
type: "practice"
difficulty: "Deep"
---

# How do you reason about the correctness boundary between a Lambda architecture's batch normalized layer and its speed denormalized layer?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Denormalization in Lakes](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Lambda architecture, the batch normalized layer handles complex, time-consuming operations like full data normalization over historical data, while the speed denormalized layer deals with real-time updates and immediate queries. The key challenge is ensuring that these two layers are correctly synchronized so that they provide consistent, up-to-date data to consumers. This question delves into the nuances of maintaining this synchronization across a normalized batch processing pipeline and a denormalized stream-processing system.

#### Triage

The Lambda architecture consists of three main components: the batch layer (which processes historical data in full batches), the speed layer (which provides near-real-time updates using streaming data), and the union layer (which merges results from both layers to provide a complete view). The question focuses on the correctness boundary between these two layers, specifically how to ensure that denormalized data in the speed layer does not diverge from the normalized data produced by the batch layer.

#### First-Principles Solution

Understanding Normalization and Denormalization:

Normalization: Ensures data consistency over time by enforcing schema constraints and removing redundancy.
Denormalization: Optimizes read performance by duplicating or storing derived data to reduce join operations, which can be expensive in real-time systems.

Batch Layer Workflow:

The batch layer processes historical data incrementally, ensuring that the output is fully normalized. This step can take significant time and resources but guarantees correctness over long periods.

#### Example
A full ETL process that updates a denormalized table based on historical records.

Speed Layer Workflow:

The speed layer uses incremental or stream processing to handle real-time data streams, updating the denormalized data immediately as new events occur.

#### Example
A Kafka stream processor that updates a denormalized database with real-time event data.

Synchronization Mechanisms:

Batch-First Approach: The batch layer operates first, and its output is used to update the speed layer’s state periodically (e.g., every 5 minutes).
Speed-First Approach: The speed layer's updates are initially applied immediately, but periodic batch runs check for any inconsistencies and correct them.
Detailed Reasoning

Batch First:

Process Order: The batch layer processes historical data first to ensure consistency.
Data Freshness Trade-off: Real-time queries rely on the speed layer’s immediate updates, which may not always reflect the most recent state due to latency.
Periodic Synchronization: A periodic batch run checks and corrects any divergences between the batch and speed layers.

Speed First:

Immediate Updates: The speed layer provides real-time updates, making it more responsive to queries.
Incremental Corrections: Periodic batch runs incrementally correct any inconsistencies detected in the speed layer’s data.
Risk of Inconsistencies: There is a risk that the denormalized state in the speed layer can diverge significantly from the normalized state.

#### Closing Summary

In summary, the Lambda architecture requires careful design to ensure that the batch and speed layers remain synchronized. The choice between Batch First or Speed First depends on the specific use case, with trade-offs between data freshness and consistency. A periodic synchronization mechanism is crucial to maintain correctness over time.

#### Common Mistakes/Follow-Ups
Ignoring Periodic Synchronization: Failing to regularly check for and correct divergences can lead to significant data inconsistencies.
Over-reliance on Speed Layer: Relying solely on real-time updates without periodic batch corrections can result in a system that becomes too complex and error-prone.
Inadequate Testing and Monitoring: Lack of proper testing frameworks and monitoring tools can make it difficult to detect and correct issues.

#### Example Scenario

#### Cons
ider an e-commerce application where order data needs to be processed for analytical purposes. The batch layer would process historical orders, ensuring that all transaction details are normalized and stored in a consistent manner. The speed layer would update the same dataset with real-time information like new orders or changes in order status. A periodic batch run can check these updates against the original source of truth, ensuring no discrepancies.

**Comparison:**

```
Data Layer	Processing Type	Source	Update Frequency
Batch	Incremental	Historical Orders	Periodic (e.g., every 5 minutes)
Speed	Real-time	New Transactions and Events	Immediate
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Batch Layer] --> B1(Incremental Historical Processing);
    B1 --> C[Normalized Data Storage];
    A --> D[Periodic Synchronization];
    D --> E[Speed Layer] --> F(Real-time Updates);
```

**Example input / output:**

| Batch Layer | Speed Layer |
|-------------|------------|
| Historical Data Processing | Real-time Updates |
| Periodic Synchronization | Immediate Data Updates |

**Data:**

```json
{
  "batchLayer": {
    "processingType": "Incremental",
    "dataSource": "Historical Orders"
  },
  "speedLayer": {
    "processingType": "Real-time",
    "dataSource": "New Transactions and Events"
  }
}
```

**Flow:**

1. Batch Layer
2. Increment Historical Data
3. Normalize Data
4. Store in Database
5. Periodic Sync
6. Synchronize Speed Layer
7. Speed Layer
8. Update Real-time Data

