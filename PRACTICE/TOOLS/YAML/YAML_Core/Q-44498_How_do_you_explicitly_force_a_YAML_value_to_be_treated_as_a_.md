---
type: "practice"
difficulty: "Deep"
---

# How do you explicitly force a YAML value to be treated as a string using a tag?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Quoting and Scalars](<../../../../TOOLS/YAML/YAML_Core/Quoting_and_Scalars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a YAML value is treated as a string, we can use the !!str tag before the value. This explicit tagging tells the YAML parser to interpret this particular value strictly as a string, even if it looks like another type (e.g., an integer or boolean).

For example:

YAML
message: !!str "This should be treated as a string"

In this case, !!str is used to explicitly indicate that "This should be treated as a string" must remain as a string.

#### Triage
The question asks about forcing a YAML value to be treated as a string using a tag. We need to understand that the primary purpose of tags in YAML is to provide explicit type information, especially when the default parsing might not meet our expectations.

#### Assumptions

The reader understands basic YAML structure.
They are familiar with how YAML parses values without tags.
They know about common types like strings, integers, and booleans.

#### First-principles solution
YAML is a data serialization language that can represent structured data as key-value pairs. By default, the parser interprets values according to their literal form, which often results in automatic type conversions (e.g., 1 becomes an integer, "true" becomes a boolean). However, there are situations where we need more control over the interpretation of certain values.

To force a value to be treated as a string, use the !!str tag. Here's why:

Why !!str? It stands for "string" and is used by the YAML specification to enforce that the following literal text should be treated strictly as a string.
Use cases: This can be useful when you want to store raw strings, such as passwords or complex data structures (e.g., JSON inside a YAML file).

Step-by-step reasoning:

Define the value: In our example, we have the string "This should be treated as a string".
Apply the !!str tag: Place !!str immediately before the value to explicitly mark it.
Parse: When the YAML parser encounters this tagged value, it will treat the entire literal text as a string.

#### Closing summary
By using !!str, we can ensure that certain values are not automatically converted by the YAML parser, maintaining their intended type and representation.

#### Common mistakes/follow-ups

Mistake 1: Not understanding that tags like !!str change how the value is interpreted.

#### Follow-up
question: What if you want to use a boolean inside this string? For example, should "true" be treated as a boolean or a part of the string?

FIELD:code_blocks>>>

YAML
message: !!str "This should be treated as a string"

None

None

None

None

Correctness and completeness confirmed.

