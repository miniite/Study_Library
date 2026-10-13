---
type: "practice"
difficulty: "Expert"
---

# You need to publish an immutable snapshot of a large object that readers use lock-free while a writer prepares the next one. Design the safe-publication pattern and explain why immutability + a single volatile reference is enough.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lock-free Read Paths](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Lock-free_Read_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design the safe-publication pattern, we need to ensure that readers can access an immutable snapshot of a large object without locking while writers prepare the next version. The key idea here is using immutability and a single volatile reference to achieve this concurrency safely.

The pattern works by having a writer prepare a new immutable snapshot in memory and then updating a single volatile reference to point to it, ensuring that readers see only valid snapshots. Let’s break down the solution step-by-step:

Immutable Snapshots: Ensure that each snapshot is immutable once created. This prevents any accidental changes after being published.
Single Volatile Reference: Use a single volatile reference (e.g., volatile Object* in C++ or AtomicReference<Object> in Java) to manage the current version of the object. A volatile reference ensures visibility and ordering guarantees, which are crucial for concurrency.

#### Triaging the Question

The interviewer is probing:

Understanding of concurrency patterns.
Knowledge of race conditions and how to avoid them.
Familiarity with immutability and its benefits in concurrent systems.
Ability to design a pattern that ensures consistency and correctness under multi-threaded access.

#### Assumptions
include:

The system supports atomic operations for updating the volatile reference.
Readers will not see an incomplete snapshot (i.e., garbage or inconsistent state).

#### First Principles Solution
Immutable Snapshot Preparation: When writing, create a new version of the object but keep it isolated until completion.
Update Volatile Reference Atomically: Once the new snapshot is ready and fully formed, update the volatile reference atomically to point to this new object.
Key Steps:
Writer: Prepare the new snapshot in a separate memory location. Ensure no writes are made directly into the current published snapshot.
Volatile Update: Perform an atomic update of the volatile reference to point to the newly created snapshot.
Reader: Always read through the volatile reference, ensuring that it points to a valid and immutable version.
Detailed Solution
Writer Thread:
Create a new snapshot in a temporary location (e.g., heap).
Ensure all writes are completed without interference from readers by using locks or other synchronization mechanisms.
Once complete, update the volatile reference atomically:
JAVA
// Pseudocode for updating the volatile reference
currentSnapshot = volatileReference; // Read current snapshot
newSnapshot = prepareNewSnapshot(currentSnapshot); // Create new immutable version
volatileReference = newSnapshot; // Update the reference (atomic)
Reader Thread:
Access the object only through the volatile reference, ensuring that it always points to a valid snapshot.
Since the reference is volatile, changes are visible across threads immediately.

#### Closing Summary
and Common Mistakes

The safe-publication pattern ensures that readers see only complete snapshots while writers prepare new ones. The use of immutability prevents any race conditions or inconsistent states during updates. A single volatile reference guarantees that updates are atomic and visible to all readers.

#### Common Mistakes

Not making the snapshot immutable, leading to potential race conditions.
Failing to perform the update atomically, risking partial updates.
Ignoring thread visibility, where readers might see stale or partially constructed snapshots.

#### Example
in Java

Here’s a simple example using AtomicReference for managing the volatile reference:

JAVA
import java.util.concurrent.atomic.AtomicReference;

public class SnapshotPublisher {
    private final AtomicReference<MyLargeObject> currentSnapshot = new AtomicReference<>(null);

    public void write(MyLargeObject oldSnapshot, MyLargeObject newSnapshot) {
        // Ensure the new snapshot is fully prepared
        while (!currentSnapshot.compareAndSet(oldSnapshot, newSnapshot)) {
            // If the update failed, retry until successful
        }
    }

    public MyLargeObject read() {
        return currentSnapshot.get();
    }
}
Summary

The safe-publication pattern leverages immutability and a single volatile reference to ensure that readers always see complete and consistent snapshots while writers prepare new versions. This approach is crucial for maintaining correctness in multi-threaded environments.

