---
type: "practice"
difficulty: "Deep"
---

# How do you architect the boundary between the operational plane (event log / OLTP) and the analytical/feature plane so ML consumption doesn't couple them into a distributed monolith? Where does CDC-driven feature freshness pull the two too tightly together?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Central vs Domain Feature Modeling](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Central_vs_Domain_Feature_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect the boundary between the operational plane (event log or OLTP) and the analytical/feature plane in an ML context, we need to ensure that these planes remain decoupled while still enabling efficient data flow and feature freshness for machine learning models. The key is to leverage a robust data architecture pattern like Data Mesh or a combination of data lakes and warehouses, ensuring that each plane operates independently yet collaboratively.

The main challenge here is maintaining the separation between operational and analytical workloads without compromising on performance or data freshness. Let's break down this problem step-by-step:

#### Triaging the Question
What the Question is Asking: How to maintain a clean separation between transactional/operational data (event logs, OLTP) and analytical data used for ML models while ensuring feature freshness.
Key Aspects to Probe:
Data governance and ownership.

#### Performance considerations
in real-time vs. batch processing.
How to handle change data capture (CDC).
Ensuring that the operational plane is not tightly coupled with the analytical plane.

#### Assumptions
We are dealing with a large-scale system where performance and scalability are critical.
The ML models require fresh features for training and inference.
There is a need for efficient data flow between OLTP systems and analytics/ML platforms.

#### First Principles Solution
Data Mesh: A Data Mesh architecture can help by decoupling data production from consumption, ensuring that each team owns its domain of data but can access it as needed without tightly coupling systems.
Data Lakes & Warehouses: Utilize a combination of data lakes for raw event logs and structured warehouses for processed analytical data. This separation ensures that operational and analytical teams work in isolation yet share the same data ecosystem.
Change Data Capture (CDC): Implement CDC mechanisms to ensure real-time feature freshness without tightly coupling systems. Tools like Apache Kafka, AWS Kinesis, or database-specific CDC solutions can be used.
Step-by-Step Solution
Data Production:
Operational Plane (OLTP): Capture transactional data in a database or event log system.
CDC Mechanism:
Implement CDC to capture changes from OLTP and push them into a central event pipeline, e.g., Kafka topic.
Data Ingestion & Processing:

#### Cons
ume the CDC events using an event-driven architecture (EDA) pattern with microservices or batch jobs that process the data in near real-time.
Data Storage:
Store processed data in a data lake for raw material and in a structured warehouse (e.g., Snowflake, Redshift) for analytical queries.
Feature Freshness Pull:
Use data mesh principles to ensure that the ML team can access fresh features without tight coupling with OLTP systems.

#### Example
Workflow
Operational Data: A user transaction is captured in an OLTP database.
CDC Event: The database triggers a CDC event and sends it to Kafka.
Event Consumption: Microservices or batch jobs consume the events from Kafka, process them, and store them in both data lake (raw) and warehouse (processed).
Feature Freshness: ML models pull features directly from the structured warehouse.

#### Closing Summary

By leveraging a Data Mesh architecture and combining data lakes with structured warehouses, we can effectively decouple the operational plane from the analytical/ML plane while ensuring feature freshness. The key is to use CDC mechanisms to enable efficient data flow without tight coupling, thus maintaining system performance and scalability.

#### Common Mistakes
& Follow-Ups
Mistake: Tight coupling between OLTP and ML pipelines, leading to performance bottlenecks.

#### Follow-Up
Discuss how to handle complex transformations during feature engineering in a distributed environment.
Mistake: Ignoring data governance and ownership, which can lead to data inconsistencies.

#### Follow-Up
Explain the importance of data lineage and how it is maintained in this architecture.

**Comparison:**

```
CDC Mechanism	Description	Example
Kafka	Real-time event capture	kafka-console-consumer --topic transaction-changes
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[OLTP Transaction] --> B{Capture CDC};
    B -- Kafka Topic --> C[Kafka Broker];
    C --> D[Process Data (Microservice/Batch Job)];
    D --> E[Lake Raw Storage, Warehouse Processed Storage];
    E --> F[ML Model];
```

**Example input / output:**

| Input | Output |
|-------|--------|
| OLTP Transaction | Processed Data in Warehouse, Raw Data in Lake |

**Data:**

```yaml
data_lake:
  - raw_transactions

warehouse:
  - user_features
  - transaction_details
```

**Flow:**

1. Kafka Topic
2. OLTP Transaction
3. Capture CDC
4. Kafka Broker
5. Process Data(Microservice/Batch Job)
6. Lake Raw Storage, WarehouseProcessed Storage
7. ML Model

