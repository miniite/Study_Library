---
type: "practice"
difficulty: "Basic"
---

# What does str.strip(), lstrip(), and rstrip() do, and how do they differ from replace()?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ String Immutability and Performance](<../../../../TOOLS/Python/Python_Language/String_Immutability_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand str.strip(), lstrip(), and rstrip() in Python, it's essential to grasp their primary functions and how they differ from replace(). These string methods are used to remove leading or trailing characters from a string. Meanwhile, replace() is used to replace occurrences of a substring with another substring within the same string.

#### Triaging the Question

The question asks for an explanation of three Python string methods: strip(), lstrip(), and rstrip(). It also wants to know how they differ from replace(). The interviewer might be probing your understanding of string manipulation in Python, your ability to identify differences between similar functions, and your capacity to explain these concepts clearly.

#### First-Principles Solution
str.strip(): This method removes leading and trailing whitespace characters (spaces, tabs, newlines) from a string.
lstrip(): This method removes leading whitespace or specified characters.
rstrip(): This method removes trailing whitespace or specified characters.

These methods are particularly useful for cleaning up user inputs before processing them further. replace(), on the other hand, is used to replace occurrences of a substring within the string with another substring.

Detailed Explanation

str.strip(): This function removes all leading and trailing whitespace from the input string. For example:

PYTHON
>>> s = "   Hello World!   "
>>> print(s.strip())
'Hello World!'

lstrip(): This method only removes leading characters (spaces by default, but can be customized) until it finds a character that is not part of the specified set. For example:

PYTHON
>>> s = "   Hello World!   "
>>> print(s.lstrip())
'Hello World!   '

rstrip(): This method works similarly to lstrip(), but it removes trailing characters instead. For example:

PYTHON
>>> s = "   Hello World!   "
>>> print(s.rstrip())
 '   Hello World!'
Comparing with replace()
str.replace(): This method returns a copy of the original string with all occurrences of old substring replaced by new substring. For example:
PYTHON
>>> s = "Hello World! Hello Python!"
>>> print(s.replace("Hello", "Hi"))
'Hi World! Hi Python!'

replace() operates on the entire string and replaces all instances of a specified substring, whereas strip(), lstrip(), and rstrip() operate at the beginning or end of the string to remove characters. Importantly, strip() can be called with an argument to specify which characters to strip:

PYTHON
>>> s = "##Hello World!##"
>>> print(s.strip("#"))
'Hello World!'

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, str.strip(), lstrip(), and rstrip() are specialized methods for removing leading or trailing characters from strings, while replace() is used to replace occurrences of a substring. Always consider the context and requirements before choosing the appropriate method.

#### Common mistakes
include using replace() where strip() or its variants would suffice, or vice versa. It's also important to understand that these methods do not modify the original string but return a new string with the specified changes.

#### Follow-up
questions might explore edge cases like handling multi-character stripping or discussing performance implications in large strings.

