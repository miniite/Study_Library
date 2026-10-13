---
type: "practice"
difficulty: "Deep"
---

# Your Prototype-based cloning does a shallow copy and two 'independent' objects share a mutable list, causing a nasty prod bug; how do you find it and design a clone contract that's safe by default?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Domain Event Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/Domain_Event_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of shared mutable state in prototype-based cloning leading to unexpected behavior (as seen in your prototype-based shallow copy), we need to ensure that each object has its own independent copies of any mutable data structures. This involves designing a clone contract and implementing it correctly.

#### Triage

The question is probing how to identify issues related to shallow copying and how to design an immutable cloning mechanism that prevents shared state between objects, which can lead to unexpected bugs in production. The key assumptions are:

We're dealing with prototype-based object creation.
We need a deep copy mechanism rather than a shallow one.
The objective is to ensure all mutable data structures (like lists) are cloned independently.

#### First Principles Solution
Understand the Problem: Identify that the issue arises from shallow copying, where shared references can lead to unintended modifications in objects.
Design a Clone Contract: Define an interface or abstract base class that ensures deep cloning of any mutable data structures.
Implement the Deep Cloning Logic: Ensure that all fields are recursively cloned to prevent shared state.
Step-by-Step Solution

Identify the Issue:

In prototype-based cloning, objects may share references to mutable data structures like lists or maps, leading to unintended side effects when one object modifies its internal state.

Define a Clone Contract:

Create an interface DeepCloneable with methods for deep cloning.
JAVA
public interface DeepCloneable {
    T deepCopy();
}

Implement the Interface:

For each mutable data structure, override the deepCopy() method to ensure a fresh copy is created.
JAVA
public class MyClass implements DeepCloneable {
    private List<String> myList;

    // Constructor and other methods...

    @Override
    public MyClass deepCopy() {
        return new MyClass(
            new ArrayList<>(myList)  // Shallow copy, needs to be deep copied
        );
    }
}

Ensure Deep Cloning for All Data Structures:

For more complex data structures (like nested objects), ensure that all mutable fields are also cloned recursively.
JAVA
public class MyClass implements DeepCloneable {
    private List<NestedObject> myNestedObjects;

    @Override
    public MyClass deepCopy() {
        return new MyClass(
            new ArrayList<>() {{
                for (NestedObject obj : this.myNestedObjects) {
                    add(obj.deepCopy());  // Ensure recursive cloning
                }
            }}
        );
    }

    public class NestedObject implements DeepCloneable {
        private String nestedField;

        @Override
        public NestedObject deepCopy() {
            return new NestedObject(nestedField);
        }
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
By defining a DeepCloneable interface and implementing it for all mutable fields, we ensure that each object has its own independent copy of any shared data structures. This approach prevents unintended side effects due to shared state.

#### Common Mistakes/Follow-Ups

Failing to recursively clone nested objects.
Assuming shallow copying is sufficient when dealing with mutable data structures.
Not testing the deep cloning mechanism thoroughly, especially edge cases involving circular references or complex data structures.

#### Example
Use Case
JAVA
public class TestDeepClone {
    public static void main(String[] args) {
        MyClass obj1 = new MyClass();
        obj1.myList.add("Item1");

        MyClass clonedObj = obj1.deepCopy();  // Clones all fields, including the list

        System.out.println(clonedObj.myList);  // Should be a separate copy
    }
}

