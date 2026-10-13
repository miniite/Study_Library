---
type: "practice"
difficulty: "Basic"
---

# Probe: why did you make this field private with a getter instead of just public? What does it buy you?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tenant Isolation in Repositories](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing software, especially in backend systems where security and encapsulation are paramount, making fields private with getters is a common practice. This approach buys us several advantages, including better data integrity, easier maintenance, and improved security. By exposing the field only through a getter method, we control how it can be accessed and manipulated from outside the class, ensuring that changes to the internal state of an object are managed in a consistent and controlled manner.

Firstly, encapsulation allows us to maintain strong data integrity by controlling access to fields. For example, if a field is marked as private with a getter, we can ensure that it's only accessed or modified through methods within the class. This helps prevent unintended modifications from external code, which could lead to bugs and security vulnerabilities.

Secondly, encapsulation promotes better maintenance of the codebase. If we find that a certain piece of data needs to be changed or restricted in some way, all we need to do is modify the getter method rather than searching through potentially numerous places where the field might have been accessed directly. This makes refactoring and updating easier.

Lastly, encapsulation enhances security by limiting exposure of sensitive data. In backend systems, it's crucial to keep certain information hidden from external access. By using getters and setters, we can add logic that prevents unauthorized users or processes from accessing private fields, thus reducing the risk of security breaches.

In summary, while making a field public might seem more convenient in terms of direct access, encapsulating it with a getter provides numerous benefits, including better data integrity, easier maintenance, and improved security. These advantages are particularly critical in complex backend systems where robustness and reliability are essential.

**Code:**

```java
public class ExampleClass {
    private int privateField;

    public int getPrivateField() {
        return privateField;
    }

    // Setter for additional logic or validation if needed
    public void setPrivateField(int privateField) {
        this.privateField = privateField; // Add validation, logging, etc.
    }
}
```

