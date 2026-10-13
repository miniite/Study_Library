---
type: "practice"
difficulty: "Basic"
---

# What is the difference between VARCHAR and NVARCHAR in SQL Server?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date and Time Types](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Date_and_Time_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

VARCHAR and NVARCHAR are both data types used to store character strings in SQL Server, but they differ fundamentally in how they handle text storage.

Firstly, TL;DR: VARCHAR stores Unicode non-unicode characters (like ASCII), while NVARCHAR stores Unicode characters, which can represent a much wider range of languages and symbols. This difference is crucial for applications that need to store multilingual data or special characters.

#### Triage
What the Question Is Asking

The question is asking about the fundamental differences between two SQL Server data types. The interviewer is probing your understanding of character encoding, storage requirements, and implications for application development, especially in scenarios involving non-ASCII text.

#### First Principles Solution

VARCHAR Data Type:

VARCHAR stands for Variable Character.
It stores a variable-length string of up to 8000 characters (for VARCHAR(MAX)).
VARCHAR stores only the character data and does not store any trailing spaces as part of the data, which means no space is wasted on unnecessary characters.

#### Example
VARCHAR(50) can hold up to 50 non-unicode characters.

NVARCHAR Data Type:

NVARCHAR stands for National Variable Character.
It stores Unicode character data, allowing it to store a wide range of languages and special characters.
NVARCHAR also supports variable-length strings but can store up to 4000 characters (for NVARCHAR(MAX)).

#### Example
NVARCHAR(50) can hold up to 50 unicode characters.
Detailed Explanation
Storage Differences
VARCHAR uses a single byte per character for ASCII data, making it more space-efficient but limiting its capability to store non-ASCII characters.
NVARCHAR requires two bytes per character, which significantly increases storage needs. However, this is necessary to support a broader range of languages and symbols.

#### Performance Considerations
VARCHAR operations can be faster due to the lower storage overhead. This makes it more suitable for applications where performance is critical and non-ASCII data are not required.
NVARCHAR might introduce slightly higher CPU usage during processing but offers greater flexibility and compatibility with international characters.

#### Common Mistakes
and Follow-Ups
Forgetting Storage Costs: Developers often overlook the increased storage cost when using NVARCHAR, which can lead to inefficient database design.
Ignoring Unicode Support: Failing to choose an appropriate data type for applications that require support for multiple languages or special characters can result in data loss or misrepresentation.

#### Closing Summary

In summary, VARCHAR and NVARCHAR serve different purposes based on the text's character encoding requirements. Choosing the right type depends on whether your application needs to store non-ASCII text, with NVARCHAR being necessary for multilingual support at a cost of increased storage space.

**Code:**

```sql
-- Example usage:
CREATE TABLE SampleTable (
    Id INT PRIMARY KEY,
    NormalText VARCHAR(50),
    UnicodeText NVARCHAR(50)
);

INSERT INTO SampleTable (Id, NormalText, UnicodeText) VALUES (1, 'Hello', N'你好');
```

