---
type: "practice"
difficulty: "Basic"
---

# How do you represent a list (sequence) of items in YAML? Give the block style syntax.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ YAML and JSON](<../../../../TOOLS/YAML/YAML_Core/YAML_and_JSON.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: In YAML, representing a list (sequence) of items is straightforward using the block style. The key idea is to use indented lines under a key, where each line starts with the same amount of indentation.

YAML (YAML Ain't Markup Language) is a data serialization language used for storing and transporting data. It allows you to represent various data structures like lists, maps, strings, numbers, booleans, and more. When it comes to representing a list in YAML, there are two common styles: the flow style and the block style.

In this question, we focus on the block style syntax, which is particularly useful for longer or complex items. Here’s how you represent a list of items using the block style:

Start with the key followed by a colon (:) and a space.
Indent each subsequent line to form the list items, ensuring that all lines have the same level of indentation.

Let's break this down step-by-step from first principles.

#### Triaging the Question

The question is asking for the block style syntax to represent a list in YAML. The interviewer might be probing your understanding of basic YAML syntax and your ability to write structured data correctly. Key assumptions include:

You understand that YAML is used for storing and transporting data.
You are familiar with different styles of representing lists.

#### First-Principles Solution

To illustrate the block style, let's consider a simple example where we represent a list of fruits:

YAML
fruits:
  - apple
  - banana
  - cherry

Here’s how to reason through this from first principles:

Identify the Key: The key is fruits, which represents the list name.
Colon and Indentation: After the key, a colon (:) is used followed by a space. This separates the key from its value.
Block Style List Items: Each item in the list starts with - (minus sign), followed by a space and then the actual data. These lines are indented to be at the same level.

#### Closing Summary

In YAML, lists (sequences) can be represented using the block style syntax where each list item is prefixed with - and follows the key colon (:). The items should have the same indentation level for clarity and correctness.

#### Common Mistakes
and Follow-Ups
Incorrect Indentation: Ensure that all lines are indented consistently. Inconsistencies can lead to parsing errors.
Missing Minus Sign (-): Each list item must start with a minus sign (-) followed by a space.
Different Styles: Mention that there is also the flow style syntax, which might be useful for shorter lists.

**Code:**

```yaml
fruits:
  - apple
  - banana
  - cherry
```

