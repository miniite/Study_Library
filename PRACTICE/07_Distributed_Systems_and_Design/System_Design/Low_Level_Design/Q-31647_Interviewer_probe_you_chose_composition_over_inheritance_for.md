---
type: "practice"
difficulty: "Deep"
---

# Interviewer probe: you chose composition over inheritance for your shape hierarchy—now the interviewer asks what you lost by not using inheritance and when inheritance would actually be the right call.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Composition vs Inheritance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Composition_vs_Inheritance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a shape hierarchy, opting for composition over inheritance is a choice driven by encapsulation, flexibility, and testability. However, there are scenarios where inheritance can offer more straightforwardness and code organization, which we need to consider. This discussion will explore the trade-offs, highlight the benefits of composition in this context, and identify situations where inheritance might be preferable.

#### Triage
What is Being Probed

The interviewer is probing your understanding of the design decisions involved in structuring a class hierarchy. Specifically, they are interested in:

Understanding: Your grasp of the fundamental principles behind object-oriented design.

#### Trade-offs
The balance between composition and inheritance in terms of maintainability, extensibility, and ease of implementation.
Context Sensitivity: Situations where certain patterns might be more appropriate than others.

#### Assumptions
We are working within a language that supports both composition and inheritance (e.g., Java, C++).
The shape hierarchy includes basic shapes like Circle, Square, and their derived shapes like FilledCircle and RoundedSquare.

#### First-Principles Solution
Composition vs. Inheritance
Composition Over Inheritance
Encapsulation: By using composition, you can encapsulate behavior within the component objects without exposing internal details.
Flexibility: Derived classes are more flexible as they do not need to adhere to a rigid inheritance contract, allowing for easier modification and extension.
Testability: Compositional approaches often lead to smaller, more testable components.

However, in this specific scenario:

Code Organization: Inheritance can make the code structure simpler by directly inheriting common functionalities.
Commonality of Behavior: If many shapes share a lot of similar behavior (e.g., drawing), inheritance can avoid redundancy.

#### Performance Considerations
In some cases, virtual function calls in inheritance can introduce overhead, which composition might mitigate.
When Inheritance Might Be the Right Call
Shared Behavior: If multiple derived classes need to implement the same set of methods or properties in a similar way.
Polymorphism: Scenarios where polymorphism is crucial for the application's functionality.

#### Cons
istency and Simplicity: When the hierarchy itself provides a clear, consistent structure that aligns with the problem domain.

#### Closing Summary

While composition offers many benefits like encapsulation and flexibility, it is not always the best choice. Inheritance can simplify code organization and provide better support for polymorphism in certain scenarios. The decision should be based on the specific requirements of the project, including performance considerations, ease of maintenance, and the overall design goals.

#### Common Mistakes/Follow-Ups
Overuse of Composition: Failing to recognize when inheritance could simplify the code.
Ignoring Polymorphism Benefits: Not considering scenarios where polymorphic behavior is essential.
Neglecting Performance Overheads: Overlooking the potential performance impacts of virtual function calls in inheritance.

Would you like to discuss a specific example or explore other design patterns that might be relevant?

**Code:**

```java
// Example of Composition
class Shape {
    void draw() {}
}

class Circle implements Shape {
    @Override
    public void draw() {
        // Draw circle logic
    }
}

class FilledCircle extends Shape {
    private Shape outerShape;

    public FilledCircle(Shape shape) {
        this.outerShape = shape;
    }

    @Override
    public void draw() {
        outerShape.draw();  // Delegate drawing to the base shape
    }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph LR;
3. A[Shape] --> B[Circle]
4. B --> C[draw()]
5. A --> D[FilledCircle]
6. D --> E[draw()]
7. C:::"Delegate drawing to the base shape"
8. E:::"Compose Circle with FilledCircle"

