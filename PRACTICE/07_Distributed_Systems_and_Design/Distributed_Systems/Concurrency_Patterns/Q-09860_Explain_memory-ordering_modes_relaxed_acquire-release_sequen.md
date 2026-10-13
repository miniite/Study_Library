---
type: "practice"
difficulty: "Expert"
---

# Explain memory-ordering modes (relaxed/acquire-release/sequentially-consistent) with a concrete producer flag + data example. Which is the minimum ordering that makes the consumer safely observe the data?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Concurrent Data Structures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Concurrent_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Memory-ordering modes are crucial in concurrent programming to ensure proper interaction between threads, especially when dealing with shared variables and synchronization. In this explanation, we will explore memory-ordering modes (relaxed, acquire-release, sequentially-consistent) using a concrete example of a producer flag and data being passed to a consumer. We'll also discuss which mode is the minimum ordering that ensures safe observation by the consumer.

First-principles wise, let's start with the core idea: memory-ordering modes control when memory operations are visible across threads. A relaxed order allows minimal synchronization, while sequentially-consistent ensures strong guarantees but may introduce unnecessary overhead. Acquire-release semantics fall in between and offer a balanced approach to ensure proper visibility of updates.

In our example, we have two threads: the producer thread sets a flag and writes data, and the consumer thread waits for the flag and then processes the data. We'll use these concepts to understand how each memory-ordering mode affects the interaction between the threads.

**Comparison:**

```
Mode	Guarantees
Relaxed	No guarantees; changes might be visible in any order
Acquire-Release	Ensures visibility of write by one thread to another
Sequentially-Consistent	All operations are globally ordered
```

**Code:**

```java
// Example Java code using volatile keyword for acquire-release semantics

public class ProducerConsumer {
    private volatile boolean flag = false;
    private int data;

    public synchronized void produce(int value) {
        // The producer thread sets the flag and writes data
        flag = true; // Volatile ensures visible changes across threads
        data = value;
    }

    public synchronized int consume() throws InterruptedException {
        while (!flag) {
            Thread.sleep(1); // Wait for the flag to be set
        }
        // The consumer thread observes both the flag and data
        return data;
    }
}
```

**Example input / output:**

```text
The producer sets the `flag` to true, then writes a value into `data`. The consumer waits until `flag` is set to true before reading `data`.

### Triage

The question asks us to explain memory-ordering modes and determine the minimum ordering that ensures safe observation of data by the consumer. We need to understand:

- **Relaxed**: Least strict, where changes might not be visible across threads immediately.
- **Acquire-release**: Ensures visibility of writes by one thread to another through reads.
- **Sequentially-consistent**: Strongest guarantee, ensuring all operations are globally ordered.

We will analyze the example using these modes and determine which provides the minimum safe ordering for the consumer.
```

**Flow:**

1. No
2. Yes
3. Producer sets flag=true
4. Is relaxed?
5. Consumer reads data withoutguarantee
6. Consumer reads data withsome guarantees
7. Consumer may miss updates
8. Consumer consistentlyobserves updated state

