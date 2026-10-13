---
type: "practice"
difficulty: "Deep"
---

# You must influence a peer principal who insists on write-through everywhere for correctness, at a real latency and cost penalty. How do you change their mind without authority, using data and shared principles?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Write-through and Write-back Consistency](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Write-through_and_Write-back_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a backend architect with deep understanding of caching strategies, I know that the pushback against write-through everywhere is driven by concerns about correctness and consistency. However, this approach comes at significant latency and cost penalties that can impact our system's performance and user experience. To change my peer's mind without authority, we need to explore the trade-offs from first principles and share a common understanding of how caching can be leveraged for both correctness and efficiency.

Firstly, let’s reframe the problem: write-through everywhere is an approach where every write operation is immediately propagated to all nodes in the cache. While this ensures eventual consistency across all nodes, it introduces latency and resource overhead. Write-around, on the other hand, only updates a local cache node, allowing for faster response times but risking temporary inconsistency.

We can use the following principles:

Latency Sensitivity: The impact of additional latency must be quantifiable.
Cost Efficiency: Caching is about reducing the load and improving throughput; write-through everywhere may not align with this goal.

#### Cons
istency Needs: Not all parts of an application require the same level of consistency.

To illustrate these points, let’s consider a scenario where we have a high-traffic e-commerce platform handling millions of transactions per day. Each transaction involves updating user data or product information in multiple cache nodes. If we implement write-through everywhere, each write operation might take several milliseconds longer due to network latency and processing overhead, which can significantly degrade the user experience.

Let’s break this down step-by-step:

Assessment: Start by understanding the current state of our caching system. Are there specific areas where consistency is critical? For example, order statuses or real-time stock updates.
Metrics Collection: Implement A/B testing to measure the impact of write-around on consistency and performance. Track metrics such as:
Latency of read and write operations
Response times for user interactions
Error rates due to temporary inconsistency
Case Study: Use a real-world example from another company that successfully implemented a hybrid approach (write-through/write-around). For instance, Amazon’s DynamoDB uses a multi-master architecture where writes are propagated asynchronously but eventually consistent.
Risk Analysis: Conduct a risk analysis of the potential failure modes for write-around versus write-through everywhere:
In write-around, temporary inconsistency could lead to read-after-write anomalies.
In write-through, the overhead and latency might not be justified if consistency is less critical.

By presenting these arguments with data-driven insights, we can demonstrate that a more nuanced approach—such as using a hybrid caching strategy—is both feasible and beneficial. This approach would allow us to maintain high performance while ensuring necessary levels of correctness without incurring unnecessary costs.

In conclusion, the key takeaway is that caching strategies should be tailored to meet specific business needs rather than applying a one-size-fits-all solution like write-through everywhere. By aligning our strategy with these principles and sharing real-world examples, we can make a compelling case for adopting a more efficient approach.

**Example input / output:**

```text
Scenario 1:
Input: User updates a product price in the database.
Output:
- Database writes new price.
- Local cache node updates price (write-around).
- Remote cache nodes eventually update price (asynchronous write-through).

Scenario 2:
Input: User requests current product price.
Output:
- Read from local cache.
- If remote caches are stale, read from database and update all caches.

Metrics Collected:
- Latency of write operations: 10ms vs. 30ms
- Response times for user interactions: 50ms vs. 70ms
- Error rate due to temporary inconsistency: 2% vs. 0%
```

