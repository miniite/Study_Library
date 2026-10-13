---
type: "practice"
difficulty: "Expert"
---

# A domain publishes a data product with a 99.9% freshness SLA but a critical downstream product needs 99.99%. How do you handle SLA composition and dependency risk across a chain of data products in a mesh?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product SLAs](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Data_Product_SLAs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this scenario, we need to design a robust data architecture that ensures end-to-end SLA (Service Level Agreement) compliance while managing dependencies between different data products. The key idea is to leverage a combination of real-time streaming and batch processing systems, along with appropriate data mesh strategies, to balance latency and freshness requirements.

#### Triage

The interviewer is probing our understanding of how to handle complex SLAs in a data-driven organization, especially when dealing with critical dependencies between different services. We need to consider the following assumptions:

The domain publishes a data product that must meet a 99.9% freshness SLA.
A downstream product has stricter requirements and demands a 99.99% freshness SLA.
There is a risk of dependency chain failure if one component does not meet its SLA.

#### First Principles Solution
Data Mesh Architecture: Implement a data mesh architecture to decouple the products while ensuring they can share data. This involves creating separate domains for each data product, with clear ownership and governance over the data.
Real-Time Processing Pipeline: Use Apache Kafka or AWS Kinesis for real-time data streaming to ensure that critical downstream products get fresh data as soon as possible.
Batch Processing Pipeline: Leverage batch processing tools like Apache Spark or AWS Glue to handle periodic, large-scale data transformations and validations.
SLA Management: Define clear SLAs and monitoring mechanisms to track the performance of each component in the chain.
Step-by-Step Solution

Define Data Domains:

Create separate domains for the domain publishing the initial product (let's call it Product A) and the downstream critical product (Product B).

Real-Time Streaming Layer:

Use Apache Kafka or AWS Kinesis to ingest raw data from multiple sources.
Implement a real-time processing layer using frameworks like Apache Flink or AWS Lambda to process the data in near real-time.

Batch Processing Layer:

Set up batch processing pipelines for periodic, intensive operations such as ETL (Extract, Transform, Load) and validation.
Use tools like Apache Spark or AWS Glue for complex transformations and validations.

Data Warehouse/Lake:

Store the processed data in a central data warehouse or lake using technologies like Amazon Redshift, Google BigQuery, or Databricks Delta Lake.
Ensure that both Product A and Product B can access this shared data.

SLA Monitoring and Alerting:

Implement monitoring tools to track SLA compliance for each component in the chain.
Use Prometheus, Grafana, or AWS CloudWatch to set up alerts when any component fails to meet its SLA.

Resilience and Failover Mechanisms:

Design failover mechanisms to ensure that if one product fails, the downstream products can still operate with some level of data freshness.
Use techniques like data caching, retries, and fallback strategies to minimize downtime.

#### Closing Summary

By leveraging a combination of real-time streaming and batch processing systems within a robust data mesh architecture, we can ensure that both Product A and Product B meet their respective SLA requirements. Real-time pipelines will handle the critical freshness needs, while batch processing ensures that periodic validations are performed without affecting the overall performance.

#### Common Mistakes/Follow-Ups
Ignoring Real-Time Processing: Focusing solely on batch processing might lead to missed deadlines for real-time dependencies.
Overcomplicating the Architecture: A complex architecture with multiple pipelines can introduce additional points of failure and maintenance overhead. It's crucial to strike a balance between complexity and maintainability.
Lack of Monitoring and Alerts: Without proper monitoring, identifying and addressing SLA violations can be delayed, leading to service disruptions.

For follow-ups, you might discuss the trade-offs between different technologies, scaling strategies, and potential integration points with other systems in the organization.

