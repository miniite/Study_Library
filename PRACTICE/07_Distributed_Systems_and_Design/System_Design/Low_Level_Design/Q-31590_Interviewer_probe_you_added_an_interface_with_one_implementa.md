---
type: "practice"
difficulty: "Deep"
---

# Interviewer probe: you added an interface with one implementation — is that over-engineering? Defend it.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Over-engineering vs Simplicity](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When we design a system, adding interfaces and implementing them can indeed seem like over-engineering at first glance. However, in many cases, it's a judicious practice that enhances the overall robustness, flexibility, and maintainability of the codebase. Let me walk you through why this approach is valuable and how to justify its use.

#### Triaging the Question

The interviewer is probing whether adding an interface with one implementation (a common pattern in object-oriented design known as "programming to an interface") is necessary or merely extra complexity. The key here lies in understanding the principles of good software design, such as separation of concerns, reusability, and future-proofing.

#### Assumptions
We are working on a backend system where flexibility, maintainability, and scalability are crucial.
The interface represents a contract that ensures consistent behavior regardless of the underlying implementation.
The current implementation is straightforward but may evolve in the future.

#### First-Principles Solution

Adding an interface with one initial implementation can be justified if it introduces several benefits:

Flexibility for Future Changes: Interfaces allow you to change the underlying implementation without affecting other parts of the system that depend on it.
Enhanced Testability: By defining a clear contract, you can easily mock or stub implementations during testing.
Decoupling Components: It promotes loose coupling between components, making the codebase more modular and easier to maintain.
Future Scalability: Even if there's only one implementation now, adding another in the future is simpler with an interface already defined.

#### Example Scenario

Suppose we have a UserRepository that currently uses a database for storing user data. However, we might want to introduce a new storage mechanism like a cache or a file system in the future. By defining an IUserRepository interface:

JAVA
public interface IUserRepository {
    User getUserById(String id);
    void save(User user);
}

And implementing it with our current database method:

JAVA
public class DatabaseUserRepository implements IUserRepository {
    public User getUserById(String id) {
        // Implementation using a database query
    }

    public void save(User user) {
        // Implementation to insert or update in the database
    }
}

Now, if we want to introduce a cache-based repository:

JAVA
public class CacheUserRepository implements IUserRepository {
    @Override
    public User getUserById(String id) {
        // Check cache first; fetch from db if not found
    }

    @Override
    public void save(User user) {
        // Store in both cache and db
    }
}

This approach ensures that the rest of the system doesn't need to change, maintaining a clean separation between interface definition and implementation.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Adding an interface with one initial implementation is not over-engineering if it introduces flexibility, testability, decoupling, and scalability. It's a strategic decision that enhances maintainability and future-proofing the codebase. However, it’s crucial to balance this against simplicity and performance in immediate use cases.

#### Common mistakes
include:

Overly complex interfaces with too many methods.
Interfaces that don’t provide clear benefits over simple direct implementation.
Premature optimization leading to unnecessary complexity without clear benefits.

In follow-ups, you might be asked about trade-offs between interface-based design and simpler, more direct approaches. Be prepared to discuss the cost-benefit analysis of your decision in different contexts.

**Code:**

```

public interface IUserRepository { User getUserById(String id); void save(User user); }

public class DatabaseUserRepository implements IUserRepository { public User getUserById(String id) { // Implementation using a database query }

CODE
Copy
public void save(User user) {
    // Implementation to insert or update in the database
}

}

CODE
Copy

```java
public class CacheUserRepository implements IUserRepository {
    @Override
    public User getUserById(String id) {
        // Check cache first; fetch from db if not found
    }

    @Override
    public void save(User user) {
        // Store in both cache and db
    }
}
```

