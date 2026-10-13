---
type: "practice"
difficulty: "Deep"
---

# How do you make a singleton / expensive-resource initialization safe when many threads request it at first-touch? Compare eager init, double-checked locking, and a memoizing future so only one thread does the work.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bulkheads and Thread Pools](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of safely initializing a singleton or an expensive resource in a multi-threaded environment, we need to ensure that only one thread performs the initialization while other threads wait until the resource is ready. This can be achieved through three main patterns: eager initialization, double-checked locking, and memoizing futures. Each approach has its trade-offs in terms of performance and safety.

#### Triage

The question asks us to compare three concurrency patterns:

Eager Initialization: The singleton or expensive resource is initialized as soon as the program starts.
Double-Checked Locking: An initial check for nullity is performed outside a synchronized block, followed by a lock on the first successful initialization.
Memoizing Future: A Future object memoizes the result of an expensive computation, ensuring that only one thread initializes the resource while others await its completion.

#### First-Principles Solution

Let's break down each pattern in detail and analyze their pros and cons:

Eager Initialization
JAVA
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    public static Singleton getInstance() {
        return INSTANCE;
    }

    // Expensive initialization logic
    private Singleton() {
        initializeExpensiveResource();
    }

    private void initializeExpensiveResource() {
        // Simulate an expensive resource init
        System.out.println("Initializing expensive resource...");
    }
}

#### Pros

Initialization is done at startup, ensuring the resource is always ready.
No need for synchronization during access.

#### Cons

The initialization cost occurs even if no client uses it.
It might not scale well in distributed systems where resources are initialized on every instance.
Double-Checked Locking
JAVA
public class Singleton {
    private static volatile Singleton INSTANCE;

    public static Singleton getInstance() {
        if (INSTANCE == null) { // First check, can be moved out of the method
            synchronized (Singleton.class) {
                if (INSTANCE == null) { // Second check inside the lock
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }

    private Singleton() {
        initializeExpensiveResource();
    }

    private void initializeExpensiveResource() {
        System.out.println("Initializing expensive resource...");
    }
}

#### Pros

Reduces locking overhead by moving the first check outside the synchronized block.
Still safe with multiple threads, as only one thread initializes the singleton.

#### Cons

Complex and can be error-prone if not implemented carefully (e.g., volatile keyword required to ensure visibility).
Slightly less efficient than eager initialization due to additional checks.
Memoizing Future
JAVA
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class Singleton {
    private static volatile FutureTask<Singleton> INSTANCE_TASK;

    public static Singleton getInstance() throws ExecutionException, InterruptedException {
        if (INSTANCE_TASK == null) {
            synchronized (Singleton.class) {
                if (INSTANCE_TASK == null) {
                    INSTANCE_TASK = new FutureTask<>(new Callable<>() {
                        @Override
                        public Singleton call() {
                            return new Singleton();
                        }
                    });
                    new Thread(INSTANCE_TASK).start();
                }
            }
        }
        return INSTANCE_TASK.get().initializeExpensiveResource();
    }

    private Singleton() {
        // No initialization needed here; done in FutureTask's Callable
    }

    private void initializeExpensiveResource() {
        System.out.println("Initializing expensive resource...");
        // Simulate the actual initialization
    }
}

#### Pros

Ensures that only one thread initializes the singleton.
Efficient as subsequent requests are served by the Future object.

#### Cons

More complex implementation due to use of FutureTask.
Potential race condition if not handled properly (though rare).

#### Closing Summary

Each pattern has its own advantages and trade-offs. Eager initialization is simple but incurs an upfront cost, while double-checked locking optimizes that by reducing synchronization overhead but at the risk of complexity. Memoizing futures provide a balanced approach with minimal overhead once initialized.

#### Common Mistakes/Follow-Ups
Incorrect Use of volatile: Double-checking without volatile can lead to race conditions.
Overhead in Double-Checked Locking: The first check might still incur some locking overhead.
Thread Safety: Ensure that all operations modifying shared state are thread-safe.
Scalability Considerations: Eager initialization may not scale well, while memoizing futures can be more efficient.

**Code:**

```

public class Singleton { private static final Singleton INSTANCE = new Singleton();

CODE
Copy
public static Singleton getInstance() {
    return INSTANCE;
}

// Expensive initialization logic
private Singleton() {
    initializeExpensiveResource();
}

private void initializeExpensiveResource() {
    System.out.println("Initializing expensive resource...");
}

}

CODE
Copy

```java
public class Singleton {
    private static volatile Singleton INSTANCE;

    public static Singleton getInstance() {
        if (INSTANCE == null) { // First check, can be moved out of the method
            synchronized (Singleton.class) {
                if (INSTANCE == null) { // Second check inside the lock
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }

    private Singleton() {
        initializeExpensiveResource();
    }

    private void initializeExpensiveResource() {
        System.out.println("Initializing expensive resource...");
    }
}
JAVA
Copy
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class Singleton {
    private static volatile FutureTask<Singleton> INSTANCE_TASK;

    public static Singleton getInstance() throws ExecutionException, InterruptedException {
        if (INSTANCE_TASK == null) {
            synchronized (Singleton.class) {
                if (INSTANCE_TASK == null) {
                    INSTANCE_TASK = new FutureTask<>(new Callable<>() {
                        @Override
                        public Singleton call() {
                            return new Singleton();
                        }
                    });
                    new Thread(INSTANCE_TASK).start();
                }
            }
        }
        return INSTANCE_TASK.get().initializeExpensiveResource();
    }

    private Singleton() {
        // No initialization needed here; done in FutureTask's Callable
    }

    private void initializeExpensiveResource() {
        System.out.println("Initializing expensive resource...");
        // Simulate the actual initialization
    }
}
```

**Flow:**

1. Yes
2. No
3. Initialization Check
4. INSTANCE == null?
5. Create Singleton
6. Return Existing Singleton
7. Initialize Expensive Resource

