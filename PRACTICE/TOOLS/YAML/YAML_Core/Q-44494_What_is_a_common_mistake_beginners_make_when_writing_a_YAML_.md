---
type: "practice"
difficulty: "Intermediate"
---

# What is a common mistake beginners make when writing a YAML list under a key and how do you spot it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Indentation Pitfalls](<../../../../TOOLS/YAML/YAML_Core/Indentation_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with YAML, one of the most common mistakes beginners make is not handling lists correctly, often due to improper indentation or syntax errors. This can lead to unexpected behavior in your configurations. For instance, failing to ensure that items within a list are properly indented might result in the YAML parser combining adjacent items into a single scalar value instead of recognizing them as separate elements.

To understand this issue better and spot it, let's break down the problem from first principles. YAML is whitespace-sensitive, meaning spaces and tabs play a crucial role in defining structures like lists and mappings (key-value pairs). Misalignment can cause the YAML parser to misinterpret data, leading to configuration issues downstream.

#### Triage
What the Question is Really Asking

The question asks about a common mistake when writing a list under a key in YAML. The interviewer is probing your understanding of YAML syntax, particularly how indentation and structure are crucial for correct parsing. They also want you to demonstrate awareness of potential pitfalls that beginners might face.

#### First-Principles Solution
Step 1: Understanding Basic Syntax

YAML uses indentation with spaces or tabs (though tabs should be avoided) to define the hierarchical structure. A list under a key is represented using hyphens (-) followed by indented values.

YAML
key:
  - item1
  - item2
Step 2: Common Mistakes and Their Effects

A frequent mistake occurs when beginners do not consistently indent their items or mix up spaces and tabs. Here’s an example of incorrect indentation:

YAML
# Incorrect YAML due to inconsistent spacing
service:
- name: webapp
- url: http://localhost:8080

In this case, the service key is correctly indented, but its list items are not consistently aligned. This causes the parser to combine these into a single scalar value or misinterpret them as separate keys.

Step 3: Spotting the Mistake

To spot such mistakes, you can follow these steps:

#### Cons
istency Check: Ensure all items in the list are indented identically.
Whitespace Verification: Confirm that spaces and tabs are used consistently throughout the document (though using spaces is generally preferred).
Validation Tools: Utilize YAML validators like yamllint or online tools to catch these errors before running your configuration.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, beginners often struggle with consistent indentation and proper list syntax in YAML. Misaligned items can cause unexpected behavior, making it crucial to maintain strict adherence to whitespace rules. To avoid such issues, always use a consistent indentation style (spaces are safer) and validate your YAML documents thoroughly.

#### Common mistakes
include:

Inconsistent spacing between list elements.
Mismatching of spaces and tabs within the same document.
Overlooking validation tools that can help catch these issues early.

For further discussion, you might be asked to demonstrate this in practice or explore more complex scenarios involving nested lists or mappings.

