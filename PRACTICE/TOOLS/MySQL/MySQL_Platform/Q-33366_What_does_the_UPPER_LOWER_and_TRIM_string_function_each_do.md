---
type: "practice"
difficulty: "Basic"
---

# What does the UPPER, LOWER, and TRIM string function each do?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Charset and Collation Migration](<../../../../TOOLS/MySQL/MySQL_Platform/Charset_and_Collation_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases like MySQL, it's essential to know how to manipulate strings effectively. Today, let's dive into three useful string functions: UPPER, LOWER, and TRIM. These functions help you standardize your data by converting text to uppercase or lowercase and removing unwanted characters from the beginning or end of a string.

#### Triaging the Question

The question asks about three specific string manipulation functions in MySQL:

UPPER: Converts all alphabetic characters in a string to uppercase.
LOWER: Converts all alphabetic characters in a string to lowercase.
TRIM: Removes leading and trailing spaces from a string.

We'll start by understanding each function's syntax, behavior, and typical use cases before moving on to their implementation details.

#### First-Principles Solution
UPPER Function

The UPPER function converts all alphabetic characters in a given string to uppercase. It leaves numbers, symbols, and non-alphabetic characters unchanged.

SQL
SELECT UPPER('Hello World!') AS upper_case_string;

Output:

CODE
HELLO WORLD!
LOWER Function

The LOWER function is the opposite of UPPER. It converts all alphabetic characters in a given string to lowercase. Similarly, it leaves numbers, symbols, and non-alphabetic characters unchanged.

SQL
SELECT LOWER('Hello World!') AS lower_case_string;

Output:

CODE
hello world!
TRIM Function

The TRIM function removes leading and trailing spaces from a string. It can also remove specified characters if you pass them as an argument. By default, it trims spaces.

SQL
SELECT TRIM('   Hello World!  ') AS trimmed_string;

Output:

CODE
Hello World!

#### Common Mistakes
and Follow-Ups
Common Mistake: Forgetting that UPPER and LOWER convert only alphabetic characters. Numbers, symbols, and non-alphabetic characters remain unchanged.

#### Follow-Up
Often, questions might ask about the difference between TRIM, LTRIM, and RTRIM. Here are brief explanations:
LTRIM: Removes leading spaces from a string.
RTRIM: Removes trailing spaces from a string.

#### Closing Summary

In summary, UPPER, LOWER, and TRIM are fundamental string functions in MySQL that help you manipulate text data. Understanding these functions is crucial for preprocessing your data before storage or display to ensure consistency and accuracy.

**Code:**

```sql
-- UPPER function example
SELECT UPPER('Hello World!') AS upper_case_string;

-- LOWER function example
SELECT LOWER('Hello World!') AS lower_case_string;

-- TRIM function example
SELECT TRIM('   Hello World!  ') AS trimmed_string;
```

**Example input / output:**

```text
-- UPPER function input/output
SELECT UPPER('Hello World!') AS upper_case_string; -- Output: HELLO WORLD!

-- LOWER function input/output
SELECT LOWER('Hello World!') AS lower_case_string; -- Output: hello world!

-- TRIM function input/output
SELECT TRIM('   Hello World!  ') AS trimmed_string; -- Output: Hello World!
```

