---
type: "practice"
difficulty: "Intermediate"
---

# How do you write a multi-line string in YAML using the literal block scalar '|'? What does it preserve?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Quoting and Scalars](<../../../../TOOLS/YAML/YAML_Core/Quoting_and_Scalars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with YAML, it's essential to know how to handle multiline strings properly, especially when dealing with configuration files or data that spans multiple lines. In YAML, a literal block scalar can be used to represent multi-line strings while preserving all the whitespace and newlines within them.

#### Triaging the Question

The question asks you to write a multi-line string in YAML using the | character (known as the literal block scalar). The key point is understanding what this syntax does, which involves maintaining the exact formatting of the text. This concept is particularly useful for embedding code snippets or large blocks of text that must retain their original formatting.

#### First Principles Solution

To write a multi-line string in YAML using the | character:

Start with |: This tells YAML to interpret the following content as a literal block scalar.
Indents are Preserved: Any indentation within the block will be retained in the final output.
Newlines Are Retained: Each newline character is preserved, and it adds an extra line in the result.

Here’s how you can write such a string:

YAML

#### example
  multiline_string: |
    This is a multi-line
    string that maintains all
    its original formatting,
    including indentation.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using | in YAML ensures that your multi-line strings are formatted exactly as they appear. This is particularly useful when you need to include code snippets or configuration details where the layout matters.

#### Common Mistakes

Forgetting to use |, which would result in the content being interpreted differently.
Not understanding that indentation within the block scalar will be preserved, leading to incorrect formatting.

#### Example

Let's walk through an example:

YAML
api:
  request_body:
    |   {
    |     "name": "John Doe",
    |     "age": 30,
    |     "email": "johndoe@example.com"
    |   }

When parsed, this YAML snippet would look like the JSON object:

JSON
{
  "name": "John Doe",
  "age": 30,
  "email": "johndoe@example.com"
}

#### Conclusion

Understanding how to use literal block scalars in YAML is crucial for maintaining proper formatting and ensuring that your configuration or data retains its original structure. Always remember the | character when dealing with multi-line content in YAML files.

**Code:**

```yaml
api:
  request_body:
    |   {
    |     "name": "John Doe",
    |     "age": 30,
    |     "email": "johndoe@example.com"
    |   }
```

**Example input / output:**

```

The input YAML snippet:

YAML
Copy
api:
  request_body:
    |   {
    |     "name": "John Doe",
    |     "age": 30,
    |     "email": "johndoe@example.com"
    |   }

When parsed, it produces the JSON output:

JSON
Copy
{
  "name": "John Doe",
  "age": 30,
  "email": "johndoe@example.com"
}
```

**Data:**

```json
{
  "name": "John Doe",
  "age": 30,
  "email": "johndoe@example.com"
}
```

