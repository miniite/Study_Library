---
type: "practice"
difficulty: "Intermediate"
---

# How do generics/type parameters improve an LLD like a reusable Repository or Cache? Show a signature.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Composition vs Inheritance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Composition_vs_Inheritance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In low-level design, especially when developing reusable components like repositories or caches, leveraging generics and type parameters can significantly enhance code flexibility, maintainability, and performance. Essentially, generics allow us to create types that are parameterized by other types, enabling the creation of reusable, type-safe code. This is particularly valuable in backend development where different data models and operations might share common patterns.

#### Triage

The question probes how using generics can improve the design of a repository or cache component. The key idea here is to make these components more generic, allowing them to work with any type of entity while maintaining strong type safety.

#### First-Principles Solution
Identify the Problem: Repositories and caches often need to handle different types of entities. Without generics, this would require creating separate classes or methods for each data model.
Generic Types: Use a generic parameter T (where T is a type variable) to represent the entity type that the repository will operate on.
Type Safety and Flexibility: By making the repository generic, we ensure that all operations are type-safe and can work with any type of data model.

#### Example
Signature

Here’s how you might define a generic Repository class in Java:

JAVA
public interface Repository<T> {
    void save(T entity);
    T findById(Long id);
    List<T> findAll();
}

This signature indicates that the repository will operate on entities of type T, where T can be any concrete type. For example, you could use this repository for a UserRepository or an OrderRepository.

#### Closing Summary
and Common Mistakes

By using generics in your LLD, you achieve code that is more reusable, maintainable, and easier to understand. However, there are common mistakes:

Not Using Generics: Failing to leverage generics can lead to code duplication and less flexible components.
Ignoring Type Safety: Ensuring that the types used with generics adhere to the expected operations to avoid runtime errors.
Common Follow-Ups
How do you ensure type safety when using generics?
Can you provide an example where a generic cache might be beneficial?
What are the trade-offs of using generics in LLD?

**Code:**

```java
public interface Repository<T> {
    void save(T entity);
    T findById(Long id);
    List<T> findAll();
}
```

