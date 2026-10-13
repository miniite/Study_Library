---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between the utf8 and utf8mb4 character sets and why does it matter for emojis?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Charset and Collation Migration](<../../../../TOOLS/MySQL/MySQL_Platform/Charset_and_Collation_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with MySQL, understanding the differences between utf8 and utf8mb4 character sets is crucial, especially when dealing with emoji support. In a nutshell, utf8 can store up to four-byte characters but doesn't handle some emoji characters properly due to its limitations. On the other hand, utf8mb4 fully supports all Unicode characters, including emojis and other special symbols.

To understand this better, let's break down the question step-by-step:

#### Triage
What is Being Probed

The interviewer is likely probing your understanding of character sets in MySQL, specifically how they handle different types of data. They might also be interested in seeing if you can explain why utf8mb4 became necessary and what implications it has for database design.

#### Assumptions
We are working with a MySQL database.
The question is about the differences between utf8 and utf8mb4 character sets, particularly in relation to emoji support.

#### First Principles Solution

Character Set Basics:

utf8 is an extended form of ASCII that can represent 256 characters per byte.
utf8mb4 is a variant of the UTF-8 character encoding designed for storing Unicode data, which supports up to four bytes per code point.

Emoji Support:

Before version 5.5, MySQL supported only two-byte and three-byte characters in its utf8 character set.
Emoji symbols are represented using four-byte sequences (e.g., U+1F600 for 😊). Because utf8 could not store these, it would either truncate the emoji or corrupt the data.

Introduction of utf8mb4:

To fully support all Unicode characters, including emojis and other complex symbols, utf8mb4 was introduced.
utf8mb4 is backward-compatible with utf8, meaning that any data stored in a utf8 column can be read correctly by a utf8mb4 database.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while utf8 was sufficient for most text data, it lacked the capability to handle all Unicode characters. This is why utf8mb4 became essential, especially in applications dealing with emojis and other special symbols.

#### Common mistakes
include:

Assuming that utf8 is sufficient for storing all types of text.
Overlooking the implications of not using utf8mb4, such as data corruption or loss when storing emoji characters.
Why It Matters

Choosing between utf8 and utf8mb4 can affect your application's ability to handle modern text, including emojis. For example, if your application heavily relies on social media features that include emojis, using utf8mb4 ensures that all data is correctly stored and displayed.

#### Example Scenario

#### Cons
ider a scenario where you are building a messaging app. If you use utf8, certain emojis might not display correctly or could be missing entirely. Using utf8mb4 ensures that your users can fully enjoy the rich text experience provided by modern Unicode characters.

**Code:**

```sql
-- Example SQL to create a table with utf8mb4 character set
CREATE TABLE messages (
    id INT AUTO_INCREMENT,
    content TEXT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci,
    PRIMARY KEY (id)
);
```

