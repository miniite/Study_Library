---
type: "practice"
difficulty: "Expert"
---

# What does a compare-and-swap (CAS) actually guarantee at the hardware level, and how do you build a lock-free counter with it? Where does the ABA problem come from and how do you defend against it?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lock-free Read Paths](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Lock-free_Read_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we'll delve into compare-and-swap (CAS) instructions at their hardware level, explore how to build a lock-free counter using CAS, discuss the ABA problem, and provide a solution to mitigate it.

TL;DR:

CAS is a hardware instruction that atomically checks if the value in a memory location matches an expected value before updating it. By leveraging CAS, we can implement a lock-free counter that avoids traditional locking mechanisms. The ABA problem arises when you mistakenly consider two identical values as different versions, which can lead to race conditions and data corruption. To mitigate this, modern systems often use stronger consistency models.

#### Triaging the Question

The question is asking about the fundamental guarantees provided by CAS instructions, how to implement a lock-free counter using them, and addressing the ABA problem that can occur with such implementations. The interviewer is likely probing your understanding of concurrency patterns, race conditions, and low-level hardware mechanisms.

#### Assumptions
Basic knowledge of CPU architecture and concurrent programming.
Familiarity with memory models and synchronization primitives.
Step-by-Step Solution:

Understand CAS at the Hardware Level:

Guarantees: CAS is an atomic operation that guarantees two things:
It checks if the value in a specific memory location (e.g., old_val) matches a given expected value.
If it does, it updates the value to a new value (new_val), ensuring no other thread can interfere with this operation during the check and update process.

Implementing a Lock-Free Counter:

A lock-free counter typically uses CAS in conjunction with an atomic increment or decrement mechanism.
The basic idea is that multiple threads can atomically check the current value of the counter and then try to update it using CAS, ensuring that only one thread successfully increments (or decrements) the counter at a time.

#### Example
Implementation:

CPP
class AtomicCounter {
    int val;
    std::atomic<bool> has_changed;

    // Initialize the counter with 0
    AtomicCounter() : val(0), has_changed(false) {}

    // Increment the counter in a lock-free manner
    void increment() {
        while (true) {
            if (has_changed.load()) break; // Exit if another thread has changed it

            int expected_val = val;
            if (!has_changed.compare_exchange_weak(expected_val, true)) continue;

            val++;
            has_changed.store(false);
        }
    }

    // Decrement the counter in a lock-free manner
    void decrement() {
        while (true) {
            if (has_changed.load()) break; // Exit if another thread has changed it

            int expected_val = val;
            if (!has_changed.compare_exchange_weak(expected_val, true)) continue;

            val--;
            has_changed.store(false);
        }
    }

    // Get the current value of the counter
    int get() {
        return val;
    }
};

Addressing the ABA Problem:

The ABA problem occurs because CAS only checks if a memory location is equal to an expected value but does not distinguish between different versions of the same value.
For example, if val was 10 and later changed to 20 then back to 10, CAS would fail when checking for the original 10 again because it sees the current value as 10.

Mitigating ABA:

To defend against ABA, you can use a stronger consistency model such as atomic compare-and-swap with version numbers.
Each time a memory location is modified, increment a version number associated with that location. CAS will then check both the value and the version number.

#### Example
Implementation with Version Numbers:

CPP
class AtomicCounterWithVersion {
    int val;
    std::atomic<int> version;

    // Initialize the counter with 0
    AtomicCounterWithVersion() : val(0), version(0) {}

    // Increment the counter in a lock-free manner using version numbers
    void increment() {
        while (true) {
            int expected_version = version.load();
            int old_val = val;
            int new_val = old_val + 1;

            if (!has_changed.compare_exchange_weak(expected_version, expected_version + 1)) continue;

            has_changed.store(false);
            if (old_val == expected_val) {
                val = new_val;
            } else { // ABA condition
                version.fetch_add(1); // Increment the version number
            }
        }
    }

    // Decrement the counter in a lock-free manner using version numbers
    void decrement() {
        while (true) {
            int expected_version = version.load();
            int old_val = val;
            int new_val = old_val - 1;

            if (!has_changed.compare_exchange_weak(expected_version, expected_version + 1)) continue;

            has_changed.store(false);
            if (old_val == expected_val) {
                val = new_val;
            } else { // ABA condition
                version.fetch_add(1); // Increment the version number
            }
        }
    }

    // Get the current value of the counter
    int get() {
        return val;
    }
};

#### Closing Summary
and Common Mistakes:

#### Common Mistakes

Forgetting to use a consistent memory model that accounts for both values and versions.
Incorrect handling of ABA conditions, leading to data corruption or deadlock.

#### Follow-Ups

Discussing how different consistency models (e.g., sequential consistency vs. weak memory models) affect the implementation.
Exploring trade-offs between performance and correctness in lock-free algorithms.

**Comparison:**

```
No
Yes
Check Version and Value
Version Mismatch?
Update Counter
Merge Version Numbers
Decrement Counter
Increment Version Number
```

**Code:**

```
CPP
Copy
class AtomicCounter {
    int val;
    std::atomic<bool> has_changed;

    // Initialize the counter with 0
    AtomicCounter() : val(0), has_changed(false) {}

    // Increment the counter in a lock-free manner
    void increment() {
        while (true) {
            if (has_changed.load()) break; // Exit if another thread has changed it

            int expected_val = val;
            if (!has_changed.compare_exchange_weak(expected_val, true)) continue;

            val++;
            has_changed.store(false);
        }
    }

    // Decrement the counter in a lock-free manner
    void decrement() {
        while (true) {
            if (has_changed.load()) break; // Exit if another thread has changed it

            int expected_val = val;
            if (!has_changed.compare_exchange_weak(expected_val, true)) continue;

            val--;
            has_changed.store(false);
        }
    }

    // Get the current value of the counter
    int get() {
        return val;
    }
};

class AtomicCounterWithVersion {
    int val;
    std::atomic<int> version;

    // Initialize the counter with 0
    AtomicCounterWithVersion() : val(0), version(0) {}

    // Increment the counter in a lock-free manner using version numbers
    void increment() {
        while (true) {
            int expected_version = version.load();
            int old_val = val;
            int new_val = old_val + 1;

            if (!has_changed.compare_exchange_weak(expected_version, expected_version + 1)) continue;

            has_changed.store(false);
            if (old_val == expected_val) {
                val = new_val;
            } else { // ABA condition
                version.fetch_add(1); // Increment the version number
            }
        }
    }

    // Decrement the counter in a lock-free manner using version numbers
    void decrement() {
        while (true) {
            int expected_version = version.load();
            int old_val = val;
            int new_val = old_val - 1;

            if (!has_changed.compare_exchange_weak(expected_version, expected_version + 1)) continue;

            has_changed.store(false);
            if (old_val == expected_val) {
                val = new_val;
            } else { // ABA condition
                version.fetch_add(1); // Increment the version number
            }
        }
    }

    // Get the current value of the counter
    int get() {
        return val;
    }
};
```

**Example input / output:**

| Request | Input | Output |
|---------|-------|--------|
| Increment (Thread 1) | Expected Value: 0, New Value: 1 | Success |
| Increment (Thread 2) | Expected Value: 0, New Value: 1 | Failed (ABA Condition) |
| Decrement (Thread 3) | Expected Value: 1, New Value: 0 | Failed (ABA Condition) |

**Data:**

```json
{
  "counter": {
    "value": 0,
    "version": 0
  },
  "increment_requests": [
    { "thread_id": 1, "expected_value": 0, "new_value": 1 },
    { "thread_id": 2, "expected_value": 0, "new_value": 1 }
  ],
  "decrement_requests": [
    { "thread_id": 3, "expected_value": 1, "new_value": 0 }
  ]
}
```

**Flow:**

1. flow - A to B, B no to C, B yes to D, C to E, D to F
2. A[Check Version and Value] --> |Version Mismatch?| B{Version Mismatch?}
3. B -- No --> C[Update Counter]
4. C --> E[Decrement Counter]
5. B -- Yes --> D[Merge Version Numbers]
6. D --> F[Increment Version Number]

