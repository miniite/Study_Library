---
type: "practice"
difficulty: "Deep"
---

# A bar-raiser asks how you'd design 'a class that owns a scarce budget' (rate-limit tokens, connection slots) so borrowing and returning are symmetric and a caller that forgets to return doesn't slowly starve the pool; how do you make return automatic and detect leaks?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Lock Granularity Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a class that manages scarce resources like rate-limit tokens or connection slots, ensuring symmetry in borrowing and returning, and automatically detecting resource leaks, we need to focus on a few key principles: encapsulation, thread safety, resource tracking, and automatic return mechanisms. The core idea is to create a robust mechanism where the borrower doesn’t have to worry about returning resources, thereby preventing resource starvation or leaks.

The class in question will manage a pool of connections or tokens, ensuring that each borrowed item is eventually returned without manual intervention from the caller. We'll implement this by tracking active borrows and automatically managing the lifecycle of these resources.

#### Triaging the Question

The interviewer is probing:

Resource Management: Understanding how to efficiently and safely manage scarce resources.
Automatic Mechanisms: Implementing mechanisms that ensure resources are returned without manual effort, detecting leaks, and handling exceptions gracefully.
Thread Safety: Ensuring the solution works correctly in multi-threaded environments.

#### Assumptions
The class will be used in a multi-threaded environment where multiple threads can borrow and return resources simultaneously.
Each resource (token or connection) has a limited lifespan and should be returned to the pool after use.
We need to detect and handle leaks where resources are not returned correctly, leading to resource starvation.

#### Solution

Resource Pool Class Design:

The class will maintain a pool of resources.
It will keep track of active borrows using a BorrowState structure.
Upon borrow, the state is updated, and upon return, it is released back into the pool.

Automatic Return Mechanism:

We can use RAII (Resource Acquisition Is Initialization) techniques to ensure resources are returned automatically when they go out of scope or an exception occurs.

Thread Safety:

Synchronization mechanisms such as locks or atomics will be used to manage concurrent access.
A ConcurrentHashMap can be used for thread-safe tracking of active borrows.

Resource Leaks Detection:

Implement a mechanism to detect and log resource leaks by comparing the number of resources borrowed with those returned.

#### First-Principles Solution

Let's start with a basic class structure:

JAVA
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

class Resource {
    private final int id;
    private boolean isAvailable = true;

    public Resource(int id) {
        this.id = id;
    }

    public void use() {
        if (isAvailable) {
            System.out.println("Resource " + id + " used.");
            isAvailable = false;
        } else {
            throw new IllegalStateException("Resource already in use.");
        }
    }

    public synchronized void release() {
        System.out.println("Resource " + id + " released.");
        isAvailable = true;
    }
}

class ResourcePool {
    private final int maxSize;
    private final ConcurrentHashMap<Integer, Resource> resourceMap;
    private AtomicInteger activeCount;

    public ResourcePool(int maxSize) {
        this.maxSize = maxSize;
        this.resourceMap = new ConcurrentHashMap<>(maxSize);
        this.activeCount = new AtomicInteger(0);

        // Initialize resources
        for (int i = 0; i < maxSize; i++) {
            resourceMap.put(i, new Resource(i));
        }
    }

    public synchronized void borrow(int id) throws Exception {
        if (!resourceMap.containsKey(id)) {
            throw new NoSuchElementException("Resource " + id + " does not exist.");
        }

        // If no resources are available and we're out of budget, throw an exception
        if (activeCount.incrementAndGet() > maxSize) {
            activeCount.decrementAndGet(); // Revert increment for check
            throw new ResourceBudgetExceeded("No available resources left.");
        }

        resourceMap.get(id).use();
    }

    public synchronized void returnResource(int id) {
        Resource r = resourceMap.get(id);
        if (r != null && !r.isAvailable()) {
            r.release();
            activeCount.decrementAndGet(); // Decrement active count
        }
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we have designed a ResourcePool class that manages a pool of resources with automatic borrowing and returning. The borrow method ensures that only the available resources can be borrowed, and an exception is thrown if all resources are in use. The returnResource method releases the resource back to the pool automatically.

#### Common mistakes
include:

Not handling exceptions properly when trying to borrow a resource.
Failing to decrement the active count correctly after releasing a resource.
Overlooking thread safety in multi-threaded scenarios.

#### Follow-ups
might explore how to handle dynamic resizing of the resource pool, implementing better logging mechanisms for leaks, and optimizing performance under high concurrency. The interviewer might also ask about how this class fits into a larger system or suggest variations such as using a different data structure or adding timeouts for borrowed resources.

**Code:**

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

class Resource {
    private final int id;
    private boolean isAvailable = true;

    public Resource(int id) {
        this.id = id;
    }

    public void use() {
        if (isAvailable) {
            System.out.println("Resource " + id + " used.");
            isAvailable = false;
        } else {
            throw new IllegalStateException("Resource already in use.");
        }
    }

    public synchronized void release() {
        System.out.println("Resource " + id + " released.");
        isAvailable = true;
    }
}

class ResourcePool {
    private final int maxSize;
    private final ConcurrentHashMap<Integer, Resource> resourceMap;
    private AtomicInteger activeCount;

    public ResourcePool(int maxSize) {
        this.maxSize = maxSize;
        this.resourceMap = new ConcurrentHashMap<>(maxSize);
        this.activeCount = new AtomicInteger(0);

        // Initialize resources
        for (int i = 0; i < maxSize; i++) {
            resourceMap.put(i, new Resource(i));
        }
    }

    public synchronized void borrow(int id) throws Exception {
        if (!resourceMap.containsKey(id)) {
            throw new NoSuchElementException("Resource " + id + " does not exist.");
        }

        // If no resources are available and we're out of budget, throw an exception
        if (activeCount.incrementAndGet() > maxSize) {
            activeCount.decrementAndGet(); // Revert increment for check
            throw new ResourceBudgetExceeded("No available resources left.");
        }

        resourceMap.get(id).use();
    }

    public synchronized void returnResource(int id) {
        Resource r = resourceMap.get(id);
        if (r != null && !r.isAvailable()) {
            r.release();
            activeCount.decrementAndGet(); // Decrement active count
        }
    }
}
```

**Flow:**

1. true
2. false
3. yes
4. no
5. true
6. false
7. ResourcePool::borrow
8. isAvailable?
9. use
10. Exception
11. Increment activeCount
12. Check if out of budget?
13. Exception:ResourceBudgetExceeded
14. Return resource
15. Decrement activeCount
16. Resource::release
17. ResourcePool::returnResource
18. Get Resource from map
19. isAvailable?
20. M
21. Exception:IllegalStateException
22. Release resource
23. Decrement activeCount

