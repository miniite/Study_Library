---
type: "practice"
difficulty: "Deep"
---

# Your ELT-in-warehouse approach means all transformation logic lives in 8000 dbt models with a 6-hour DAG. As principal, how do you decide what should move out of the warehouse (to Spark/streaming) versus stay, and how do you prevent the warehouse from becoming an expensive general-purpose compute engine?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Warehouse as System of Record](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of deciding which transformations to offload from a central ELT-in-warehouse approach while preventing the warehouse from becoming an overburdened, let’s break this down step-by-step:

#### Overview
In an ELT-in-warehouse architecture, data is primarily transformed within the data warehouse itself using tools like dbt. However, as your system scales to handle more complex and compute-intensive transformations, you need a strategy to offload these tasks to other systems such as Spark or streaming platforms. This decision involves balancing resource utilization, cost efficiency, and performance.

#### Triage
The core question is about identifying which transformations can be moved out of the warehouse and onto more specialized platforms like Spark for parallel processing. We will also need to ensure that the data warehouse remains efficient by avoiding it from becoming a general-purpose compute engine.

#### First-Principles Solution

Identify Compute Intensive Transformations: Analyze which transformations are computationally heavy, such as complex aggregations, machine learning models, or real-time processing tasks.
Evaluate Data Freshness Requirements: Determine the latency requirements for different datasets and ensure that offloading only necessary computations is done to meet these needs.
Cost Efficiency Analysis: Compare the costs of running transformations in the warehouse versus other platforms. Offload tasks where there are significant cost savings or performance improvements.
Resource Utilization Monitoring: Continuously monitor resource usage within the data warehouse and identify bottlenecks that could be mitigated by offloading.

Detailed Steps:

Define Performance Metrics: Establish key performance metrics such as processing time, memory consumption, and disk I/O to measure the impact of transformations.
Create a Transformation Catalog: Document all transformation logic in dbt models and classify them based on complexity and resource usage.
Risk Assessment: Assess the risk of moving transformations. For example, real-time analytics may require immediate results and cannot tolerate delays caused by complex warehouse operations.
Parallel Processing Evaluation: Evaluate whether offloading to Spark can significantly enhance parallel processing capabilities without overburdening the warehouse.

#### Example
Decision Tree:

CODE
If (transformation is heavy AND data freshness requirements are high)
  THEN move transformation to Spark
Else if (cost savings are significant)
  THEN move transformation to a less expensive platform
Else
  KEEP transformation in the warehouse

Architecture Considerations:

Use a hybrid architecture where the warehouse handles simple and frequent transformations, while Spark processes complex and less frequent tasks.
Implement data mesh principles by distributing data processing across multiple nodes, ensuring that each component performs its specific task efficiently.

#### Closing Summary
+ Common Mistakes/Follow-Ups: By strategically offloading compute-intensive tasks to specialized platforms like Spark or streaming engines, we can optimize resource utilization and maintain the efficiency of our central warehouse. However, it’s crucial to continuously monitor and adjust this strategy based on changing workloads and technological advancements. Missteps include over-engineering the architecture, failing to leverage existing tools effectively, and neglecting data freshness requirements.

#### Common Mistakes

Over-complicating the architecture without clear benefits.
Ignoring cost savings by keeping all heavy tasks in the warehouse.
Neglecting real-time constraints when offloading complex computations.

**Comparison:**

```
Transformation Type	Resource Usage (%)	Latency Requirements
Simple Aggregation	10	High
Complex ETL	45	Medium
Real-Time Processing	75	Very Low
```

**Code:**

```
MERMAID
Copy
graph TD;
     A[Data Warehouse] --> B1{Is transformation heavy?};
     B1 -- Yes --> C[Offload to Spark];
     B1 -- No --> D[Stay in Warehouse];
     D --> E[Monitor Resource Utilization];
     F[Risk Assessment] --> C; F --> D
```

**Example input / output:**

```text
Input:
 - dbt model definitions and performance metrics for various transformations.

 Output:
 - A list of transformations to move out of the warehouse.
 - An updated data architecture diagram showing offloaded tasks.
 - Recommendations on resource allocation strategies.
```

**Data:**

```yaml
# Example Transformation Classification
 - name: "Aggregation of Daily Sales"
   type: "Semi-Complex"
   resource_usage: 30%
   latency_requirements: High

 - name: "Real-Time Fraud Detection"
   type: "Highly Complex"
   resource_usage: 70%
   latency_requirements: Very Low
```

**Flow:**

1. Yes
2. No
3. Start
4. Define Performance Metrics
5. Is Transformation Heavy?
6. Offload to Spark
7. Stay in Warehouse
8. Monitor Resource Utilization
9. Risk Assessment
10. Adjust Strategy if Needed
11. End

