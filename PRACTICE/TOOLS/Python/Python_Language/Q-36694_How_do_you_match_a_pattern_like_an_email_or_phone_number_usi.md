---
type: "practice"
difficulty: "Intermediate"
---

# How do you match a pattern like an email or phone number using the re module, and what does re.compile buy you?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String Immutability and Performance](<../../../../TOOLS/Python/Python_Language/String_Immutability_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To match patterns such as emails or phone numbers in Python, we use the re module which provides regular expression matching operations. The key idea is to define a pattern that describes the structure of what we are looking for and then compile it into a regular expression object using re.compile. This compiled object can be used repeatedly without recompiling.

First, let’s triage the question: The interviewer is probing your understanding of how to use the re module in Python effectively. They want to know that you understand both the syntax for defining patterns and why compiling a pattern might offer performance benefits.

Step-by-Step Solution
Define the Pattern: For an email, we typically look for a string with @ and ., followed by letters or digits. For a phone number, it could be a series of digits optionally separated by dashes, parentheses, or spaces.
Compile the Regular Expression: Compiling the pattern allows us to reuse the compiled object multiple times without the overhead of parsing the pattern each time.
Using re.match and re.search: These functions help in finding matches at the beginning of a string (match) or anywhere within it (search).

#### Example
Step 1: Define Patterns
PYTHON
import re

# Email pattern
email_pattern = r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'

# Phone number pattern (US format)
phone_pattern = r'\(?\d{3}\)?[-.\s]?\d{3}[-.]?\d{4}'
Step 2: Compile Patterns
PYTHON
email_regex = re.compile(email_pattern)
phone_regex = re.compile(phone_pattern)
Step 3: Use Compiled Patterns
PYTHON
text = "Contact us at john.doe@example.com or call 555-1234. Or try (987)654-3210."

# Find email
match_email = email_regex.search(text)
if match_email:
    print("Found email:", match_email.group())

# Find phone number
match_phone = phone_regex.search(text)
if match_phone:
    print("Found phone number:", match_phone.group())

#### Closing Summary
and Common Mistakes

#### Common Mistakes
One mistake might be defining overly complex or restrictive patterns, which can miss valid emails or numbers. Another is not using re.compile to optimize repeated operations.

#### Follow-Up
Questions: The interviewer might ask about performance differences between using compiled vs non-compiled regexes in loops or large datasets.
Additional Thoughts

While the re module is powerful, it’s worth noting that for very complex patterns, tools like Python's built-in string methods can be more straightforward. However, for pattern matching, re offers a concise and flexible approach.

**Code:**

```python
import re

# Email pattern definition
email_pattern = r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'
email_regex = re.compile(email_pattern)

# Phone number pattern definition (US format)
phone_pattern = r'\(?\d{3}\)?[-.\s]?\d{3}[-.]?\d{4}'
phone_regex = re.compile(phone_pattern)

text = "Contact us at john.doe@example.com or call 555-1234. Or try (987)654-3210."

# Finding email and phone number
match_email = email_regex.search(text)
if match_email:
    print("Found email:", match_email.group())

match_phone = phone_regex.search(text)
if match_phone:
    print("Found phone number:", match_phone.group())
```

**Example input / output:**

```text
Found email: john.doe@example.com
Found phone number: (987)654-3210.
```

