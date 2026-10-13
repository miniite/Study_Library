---
type: "practice"
difficulty: "Deep"
---

# You want to guarantee an endpoint stays sub-linear as data grows; what design patterns (indexing, precomputation, sharding, caps) do you reach for and how do you verify the guarantee holds?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Complexity of Retries](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_of_Retries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a backend endpoint remains efficient and scales sub-linearly as data grows, we need to focus on reducing both time complexity (how quickly it processes) and space complexity (the memory or storage required). This can be achieved through several design patterns such as indexing, precomputation, sharding, and setting limits (caps). Let's break down each of these techniques from first principles.

#### Triaging the Question

The interviewer is probing your understanding of how to handle growing data sets efficiently. They are looking for a deep understanding of time and space complexity management strategies that can be implemented in backend systems. We'll need to consider trade-offs between different solutions, ensure they meet performance guarantees, and validate these guarantees through concrete examples.

#### First-Principles Solution
Indexing: Indexes allow quick access to data by reducing the number of required lookups or traversals. For example, an index on a database column can dramatically speed up queries.
Precomputation: Compute results in advance and store them for future use. This can reduce runtime complexity at query time.
Sharding: Distribute data across multiple nodes to parallelize processing and reduce load on any single node.
Caps: Set limits or constraints on the amount of data processed by an endpoint, ensuring it does not scale disproportionately.
Verifying the Guarantee

To verify that these guarantees hold, we need to:

Measure Performance Metrics: Use tools like Prometheus for monitoring latency and throughput.
Load Testing: Simulate realistic workloads to ensure the system behaves as expected under varying conditions.
Profiling Tools: Utilize profiling tools like Valgrind or gprof to identify bottlenecks.

#### Example
Sharding

#### Cons
ider a scenario where we have an endpoint that processes user data. Initially, this could involve iterating over all users in a single database table:

PYTHON
for user in User.objects.all():
    process_user(user)

This operation has a time complexity of O(n), which is linear with the number of users.

To improve this, we can shard the users across multiple databases or partitions based on their ID. This changes our processing logic to something like:

PYTHON
shard_key = user.id % NUM_SHARDS
with connectionRouter(shard_key):
    process_user(user)

Now, each shard handles a subset of users, and the time complexity is effectively O(n/k), where k is the number of shards. This is sub-linear as long as k grows slower than n.

#### Common Mistakes/Follow-Ups
Over-Indexing: Creating too many indexes can lead to increased write times and storage costs.
Ignoring Edge Cases: Not considering what happens when a shard becomes unresponsive or when data needs to be re-balanced.
Assuming Linear Growth: Failing to account for non-linear growth patterns in user behavior, which could invalidate the sub-linear guarantee.

#### Closing Summary

In summary, ensuring an endpoint remains efficient as data grows requires a combination of indexing, precomputation, sharding, and setting appropriate caps. Each technique has its own trade-offs, but collectively they can be used to create robust systems that scale well under varying loads. Verification through performance metrics and load testing is crucial to ensure these guarantees hold in practice.

**Code:**

```python
# Example of sharding logic
def process_user(user):
    shard_key = user.id % NUM_SHARDS
    with connectionRouter(shard_key):
        # Process the user within the appropriate shard
        pass

class UserShardedManager(models.Manager):
    def get_queryset(self, **kwargs):
        return super().get_queryset().using(self.get_shard())

    @staticmethod
    def get_shard(user_id):
        return user_id % NUM_SHARDS
```

**Example input / output:**

```
Input: A user ID and the number of shards.
Output: The shard key for processing.
PYTHON
Copy
# Example input-output scenario
user_id = 5
num_shards = 2
shard_key = user_id % num_shards  # Output: 1
```

**Data:**

```json
{
  "users": [
    { "id": 1 },
    { "id": 2 },
    { "id": 3 },
    // ... many more users
  ],
  "shards": [0, 1]
}
```

