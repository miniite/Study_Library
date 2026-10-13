---
type: "practice"
difficulty: "Expert"
---

# Distinguish lock-free from wait-free algorithms. Give a scenario where lock-free is enough and one where wait-free is required, and explain the throughput/latency trade-off you accept with each.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Concurrent Data Structures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Concurrent_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In software development, concurrency patterns and race conditions are crucial to ensure data consistency and system reliability, especially in multi-threaded applications. Lock-free algorithms avoid using explicit locks (mutexes) to manage shared resources, whereas wait-free algorithms guarantee that a thread making a sequence of atomic operations will complete within a bounded number of steps. Understanding the differences between lock-free and wait-free algorithms is essential for designing robust and efficient concurrent systems.

#### Triage
What the Question is Asking

The question asks us to distinguish between lock-free and wait-free algorithms, provide scenarios where each is appropriate, and discuss the throughput/latency trade-offs involved in using these approaches. The key here is understanding that while both aim to avoid deadlocks and ensure data consistency, they do so through different mechanisms with varying performance characteristics.

#### First-Principles Solution
Scenario 1: Lock-Free Algorithm

A common scenario where a lock-free algorithm is sufficient involves scenarios requiring high throughput but not absolute latency guarantees. For example, in a producer-consumer model for a message queue or a thread-safe stack, we can use a lock-free algorithm to manage the operations without causing deadlocks. The key idea here is that while there might be occasional contention and retries due to race conditions, overall performance remains high because of reduced overhead from locking mechanisms.

#### Example
Code:

CPP
class LockFreeStack {
    int top;
    AtomicInt* data[];

    void push(int value) {
        int oldTop = top.load(std::memory_order_relaxed);
        if (oldTop < capacity) { // Ensure we have space in the stack
            data[oldTop].store(value, std::memory_order_relaxed);
            top.store(oldTop + 1, std::memory_order_release);
        } else {
            // Handle overflow condition
        }
    }

    int pop() {
        int oldTop = top.load(std::memory_order_acquire);
        if (oldTop > -1) { // Ensure there is data in the stack
            int value = data[oldTop].load(std::memory_order_relaxed);
            top.store(oldTop - 1, std::memory_order_release);
            return value;
        } else {
            // Handle underflow condition
        }
    }
};

Throughput/Latency Trade-off: In this case, we accept higher latency in the form of occasional retries and contention. However, this is balanced by reduced locking overhead and better overall throughput.

Scenario 2: Wait-Free Algorithm

A scenario where a wait-free algorithm is necessary involves critical applications requiring guaranteed bounds on thread operation time. For instance, in real-time systems or financial transaction processing, every operation must complete within a predictable time frame to meet latency requirements. This ensures that no single thread can block others indefinitely.

#### Example
Code:

CPP
class WaitFreeCounter {
    int value;
    AtomicInt* tokens[];

    void increment() {
        int token = -1; // Acquire a unique token
        while (true) {
            if ((token = tokens[__thread_id].load(std::memory_order_acquire)) >= 0) {
                break; // Token acquisition failed, retry
            }
        }

        value++;
        tokens[__thread_id].store(-1, std::memory_order_release); // Release the token

        // Critical section: Increment counter atomically
        int oldVal = value.load(std::memory_order_acquire);
        value.store(oldVal + 1, std::memory_order_release);

        // Ensure all other operations have completed before releasing the token
    }

    int decrement() {
        int token = -1;
        while (true) {
            if ((token = tokens[__thread_id].load(std::memory_order_acquire)) >= 0) {
                break;
            }
        }

        value--;
        tokens[__thread_id].store(-1, std::memory_order_release);

        // Critical section: Decrement counter atomically
        int oldVal = value.load(std::memory_order_acquire);
        value.store(oldVal - 1, std::memory_order_release);

        return token; // Return the token to reuse in future operations
    }
};

Throughput/Latency Trade-off: Here, we achieve deterministic bounds on operation latency but at the cost of increased complexity and reduced throughput due to overhead from token management.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, lock-free algorithms are effective for scenarios requiring high throughput with acceptable latencies, while wait-free algorithms ensure bounded latencies but may come at the cost of reduced throughput. The choice between them depends on the specific requirements and constraints of your application.

#### Common mistakes
include underestimating the complexity of implementing lock-free or wait-free algorithms correctly, especially in shared-memory concurrency scenarios. Always consider edge cases such as deadlock avoidance and resource exhaustion.

For follow-ups, you might be asked to discuss trade-offs further, explore other concurrency patterns like optimistic locking, or delve into implementation details for specific use cases.

**Comparison:**

```
Algorithm	Key Characteristics	Throughput	Latency
Lock-Free	High throughput with occasional retries and contention	High	Variable, but bounded by system resources
Wait-Free	Deterministic latency guarantees	Lower than lock-free	Bounded and predictable
```

**Code:**

```

class LockFreeStack { int top; std::atomic<int>* data[];

CODE
Copy
void push(int value) {
    int oldTop = top.load(std::memory_order_relaxed);
    if (oldTop < capacity) { // Ensure we have space in the stack
        data[oldTop].store(value, std::memory_order_relaxed);
        top.store(oldTop + 1, std::memory_order_release);
    } else {
        // Handle overflow condition
    }
}

int pop() {
    int oldTop = top.load(std::memory_order_acquire);
    if (oldTop > -1) { // Ensure there is data in the stack
        int value = data[oldTop].load(std::memory_order_relaxed);
        top.store(oldTop - 1, std::memory_order_release);
        return value;
    } else {
        // Handle underflow condition
    }
}

};

CODE
Copy

```cpp
class WaitFreeCounter {
    int value;
    std::atomic<int>* tokens[];

    void increment() {
        int token = -1; // Acquire a unique token
        while (true) {
            if ((token = tokens[__thread_id].load(std::memory_order_acquire)) >= 0) {
                break; // Token acquisition failed, retry
            }
        }

        value++;
        tokens[__thread_id].store(-1, std::memory_order_release); // Release the token

        int oldVal = value.load(std::memory_order_acquire);
        value.store(oldVal + 1, std::memory_order_release);

        // Ensure all other operations have completed before releasing the token
    }

    int decrement() {
        int token = -1;
        while (true) {
            if ((token = tokens[__thread_id].load(std::memory_order_acquire)) >= 0) {
                break;
            }
        }

        value--;
        tokens[__thread_id].store(-1, std::memory_order_release);

        int oldVal = value.load(std::memory_order_acquire);
        value.store(oldVal - 1, std::memory_order_release);

        return token; // Return the token to reuse in future operations
    }
};
```

**Example input / output:**

```

Input (Lock-Free Stack): push(1) push(2) pop()

Output: 1 2

CODE
Copy

```plaintext
Input (Wait-Free Counter):
increment()
decrement()

Output:
0
```

**Data:**

```yaml
stack_capacity: 1024
```

**Flow:**

1. No
2. Yes
3. Success
4. Failure
5. Lock-Free Stack Push
6. Is stack full?
7. Push new element
8. Handle overflow condition
9. Wait-Free Counter Increment
10. Acquire token
11. Increment counter atomically,then release token

