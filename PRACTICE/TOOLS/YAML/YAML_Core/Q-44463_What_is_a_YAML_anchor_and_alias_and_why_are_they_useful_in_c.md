---
type: "practice"
difficulty: "Deep"
---

# What is a YAML anchor (&) and alias (*) and why are they useful in configuration files?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Refactoring Helm Values](<../../../../TOOLS/YAML/YAML_Core/Refactoring_Helm_Values.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In YAML, anchors (&) and aliases (*) allow you to create reusable data structures by referencing them multiple times. This feature is particularly useful when dealing with complex configurations that need to reference similar or identical data, making the files more concise and easier to maintain.

#### Triage

The question asks about the use of & (anchor) and * (alias) in YAML, specifically how they work and why using them can be beneficial. The interviewer is probing your understanding of YAML's core features and their practical applications. We will assume that you are familiar with basic YAML syntax.

Detailed Solution
Understanding Anchors (&)

An anchor in YAML begins with the & symbol followed by a unique name, which we call the key. This key serves as an identifier for a particular piece of data within the file. Once this anchor is defined, it can be used multiple times throughout the document.

For example:

YAML
---
item: &example_item
  name: "A Sample Item"
  price: 10
Using Anchors with Aliases (*)

An alias in YAML starts with * followed by a key. This key must match an existing anchor's key for the alias to work correctly. An alias essentially refers back to the data defined by its corresponding anchor.

Here’s how we can use both:

YAML
---
item: &example_item
  name: "A Sample Item"
  price: 10

cart:
- *example_item
- *example_item

In this example, *example_item is used to reference the data defined by the anchor &example_item. This way, the same item details are reused without repetition.

Why They Are Useful
Code Duplication Avoidance: By defining an anchor and using it with aliases, you avoid repeating the same configuration multiple times. This reduces maintenance overhead and the chance of errors.
Readability: Complex configurations can become more readable when repeated data is minimized. The use of anchors and aliases makes the YAML file cleaner and easier to understand.
Flexibility: You can easily change a shared piece of data in one place, and all places that reference it will automatically update.

#### Example Scenario

Suppose you are configuring an application environment with multiple instances:

YAML
---
app:
  name: MyApp
  version: "1.0"
  config: &default_config
    log_level: INFO
    max_connections: 256

instance_1:
  *default_config
  instance_id: 1
  port: 8081

instance_2:
  *default_config
  instance_id: 2
  port: 8082

In this scenario, the &default_config anchor is used to define common configuration settings. These settings are then referenced by each instance, ensuring consistency and reducing redundancy.

#### Common Mistakes/Follow-Ups
Misusing Anchors and Aliases: Ensure that you correctly match anchors and aliases. If a key in an alias does not correspond to any existing anchor, the YAML parser will raise an error.
Overuse of Anchors/Aliases: While these features are powerful, overusing them can make the file harder to read if it becomes too abstract.

#### Closing Summary

Understanding and effectively using YAML anchors (&) and aliases (*) is crucial for managing complex configurations. By reducing code duplication and improving readability, they enhance maintainability and reduce the risk of errors in your configuration files.

Self-Check

Correctness + completeness confirmed.

**Code:**

```

item: &example_item name: "A Sample Item" price: 10

cart:

*example_item
*example_item
CODE
Copy
```yaml
---
app:
  name: MyApp
  version: "1.0"
  config: &default_config
    log_level: INFO
    max_connections: 256

instance_1:
  *default_config
  instance_id: 1
  port: 8081

instance_2:
  *default_config
  instance_id: 2
  port: 8082
```

